<template>
  <div class="restocking">
    <div class="page-header">
      <h2>{{ t('restocking.title') }}</h2>
      <p>{{ t('restocking.description') }}</p>
    </div>

    <div v-if="loading" class="loading">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>

      <!-- Budget Slider Card -->
      <div class="card">
        <div class="budget-label-row">
          <label class="budget-label">{{ t('restocking.budget') }}</label>
          <span class="budget-value">{{ formatCurrency(budget) }}</span>
        </div>
        <input
          type="range"
          min="1000"
          max="100000"
          step="1000"
          v-model.number="budget"
          class="budget-slider"
        />
        <div class="slider-range-labels">
          <span>$1K</span>
          <span>$100K</span>
        </div>
      </div>

      <!-- Stats Grid -->
      <div class="stats-grid">
        <div class="stat-card restocking">
          <div class="stat-label">{{ t('restocking.itemsToRestock') }}</div>
          <div class="stat-value">{{ recommendations.length }}</div>
        </div>
        <div class="stat-card info">
          <div class="stat-label">{{ t('restocking.budgetUsed') }}</div>
          <div class="stat-value">{{ formatCurrency(budgetUsed) }}</div>
        </div>
        <div class="stat-card success">
          <div class="stat-label">{{ t('restocking.budgetRemaining') }}</div>
          <div class="stat-value">{{ formatCurrency(budgetRemaining) }}</div>
        </div>
      </div>

      <!-- Success message after placing order -->
      <div v-if="orderSuccess" class="order-success">
        {{ t('restocking.orderPlaced') }}: {{ orderNumber }}
      </div>

      <!-- Recommendations Table -->
      <div class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.recommendations') }}</h3>
          <span class="badge restocking">{{ recommendations.length }}</span>
        </div>

        <div v-if="recommendations.length === 0" class="no-recommendations">
          {{ t('restocking.noRecommendations') }}
        </div>
        <div v-else class="table-container">
          <table>
            <thead>
              <tr>
                <th>{{ t('restocking.table.sku') }}</th>
                <th>{{ t('restocking.table.itemName') }}</th>
                <th>{{ t('restocking.table.currentDemand') }}</th>
                <th>{{ t('restocking.table.forecastedDemand') }}</th>
                <th>{{ t('restocking.table.demandGap') }}</th>
                <th>{{ t('restocking.table.restockQty') }}</th>
                <th>{{ t('restocking.table.unitCost') }}</th>
                <th>{{ t('restocking.table.lineTotal') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in recommendations" :key="item.item_sku">
                <td><strong>{{ item.item_sku }}</strong></td>
                <td>{{ item.item_name }}</td>
                <td>{{ item.current_demand.toLocaleString() }}</td>
                <td><strong>{{ item.forecasted_demand.toLocaleString() }}</strong></td>
                <td>
                  <span class="demand-gap">+{{ item.demandGap.toLocaleString() }}</span>
                </td>
                <td>{{ item.quantity.toLocaleString() }}</td>
                <td>${{ item.unit_cost.toLocaleString() }}</td>
                <td><strong>${{ item.line_total.toLocaleString() }}</strong></td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Place Order Button -->
        <div class="order-action">
          <button
            class="btn-primary"
            :disabled="recommendations.length === 0 || placingOrder"
            @click="placeOrder"
          >
            {{ placingOrder ? t('common.loading') : t('restocking.placeOrder') }}
          </button>
        </div>
      </div>

      <!-- Excluded Items Table (only shown when there are excluded items) -->
      <div v-if="excludedItems.length > 0" class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.excludedItems') }}</h3>
          <span class="badge warning">{{ excludedItems.length }}</span>
        </div>
        <p class="excluded-note">{{ t('restocking.excludedNote') }}</p>
        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th>{{ t('restocking.table.sku') }}</th>
                <th>{{ t('restocking.table.itemName') }}</th>
                <th>{{ t('restocking.table.currentDemand') }}</th>
                <th>{{ t('restocking.table.forecastedDemand') }}</th>
                <th>{{ t('restocking.table.demandGap') }}</th>
                <th>{{ t('restocking.table.unitCost') }}</th>
                <th>Reason</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in excludedItems" :key="item.item_sku">
                <td><strong>{{ item.item_sku }}</strong></td>
                <td>{{ item.item_name }}</td>
                <td>{{ item.current_demand.toLocaleString() }}</td>
                <td>{{ item.forecasted_demand.toLocaleString() }}</td>
                <td>
                  <!-- demandGap <= 0 means no restocking opportunity -->
                  <span :class="item.demandGap > 0 ? 'demand-gap-neutral' : 'demand-gap-negative'">
                    {{ item.demandGap.toLocaleString() }}
                  </span>
                </td>
                <td>${{ item.unit_cost.toLocaleString() }}</td>
                <td>
                  <span :class="['badge', item.demandGap <= 0 ? 'decreasing' : 'warning']">
                    {{ item.demandGap <= 0 ? 'Decreasing' : 'Over budget' }}
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted, watch } from 'vue'
import { api } from '../api'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Restocking',
  setup() {
    const { t } = useI18n()

    const loading = ref(true)
    const error = ref(null)
    const forecasts = ref([])

    const budget = ref(10000)
    const placingOrder = ref(false)
    const orderSuccess = ref(false)
    const orderNumber = ref(null)

    // Run the greedy selection algorithm as a computed property so it
    // automatically recalculates whenever budget or forecasts change.
    const allRecommendations = computed(() => {
      // Step 1: Compute demand gap per item
      const withGap = forecasts.value.map(item => ({
        ...item,
        demandGap: item.forecasted_demand - item.current_demand
      }))

      // Step 2: Separate items with positive gap (growing demand) from those without
      const increasing = withGap.filter(item => item.demandGap > 0)
      const nonIncreasing = withGap.filter(item => item.demandGap <= 0)

      // Step 3: Sort by demand gap descending — largest shortfall gets priority
      increasing.sort((a, b) => b.demandGap - a.demandGap)

      // Step 4: Greedily select items that fit within remaining budget
      let remaining = budget.value
      const selected = []
      const overBudget = []

      for (const item of increasing) {
        const lineCost = item.demandGap * item.unit_cost
        if (lineCost <= remaining) {
          selected.push({
            ...item,
            quantity: item.demandGap,
            line_total: lineCost
          })
          remaining -= lineCost
        } else {
          // Item has positive gap but cost exceeds remaining budget
          overBudget.push(item)
        }
      }

      return { selected, excluded: [...overBudget, ...nonIncreasing] }
    })

    const recommendations = computed(() => allRecommendations.value.selected)
    const excludedItems = computed(() => allRecommendations.value.excluded)

    const budgetUsed = computed(() =>
      recommendations.value.reduce((sum, item) => sum + item.line_total, 0)
    )

    const budgetRemaining = computed(() => budget.value - budgetUsed.value)

    const formatCurrency = (value) => {
      return '$' + Math.round(value).toLocaleString()
    }

    const loadForecasts = async () => {
      loading.value = true
      error.value = null
      try {
        forecasts.value = await api.getDemandForecasts()
      } catch (err) {
        error.value = 'Failed to load demand forecasts'
        console.error(err)
      } finally {
        loading.value = false
      }
    }

    const placeOrder = async () => {
      if (recommendations.value.length === 0 || placingOrder.value) return

      placingOrder.value = true
      orderSuccess.value = false
      orderNumber.value = null

      const orderData = {
        items: recommendations.value.map(item => ({
          item_sku: item.item_sku,
          item_name: item.item_name,
          quantity: item.quantity,
          unit_cost: item.unit_cost,
          line_total: item.line_total
        })),
        total_value: budgetUsed.value
      }

      try {
        const response = await api.submitRestockingOrder(orderData)
        orderNumber.value = response.order_number
        orderSuccess.value = true
      } catch (err) {
        error.value = 'Failed to place restocking order'
        console.error(err)
      } finally {
        placingOrder.value = false
      }
    }

    // Clear success message whenever the slider moves — the order context has changed
    watch(budget, () => {
      orderSuccess.value = false
      orderNumber.value = null
    })

    onMounted(loadForecasts)

    return {
      t,
      loading,
      error,
      budget,
      recommendations,
      excludedItems,
      budgetUsed,
      budgetRemaining,
      placingOrder,
      orderSuccess,
      orderNumber,
      formatCurrency,
      placeOrder
    }
  }
}
</script>

<style scoped>
.restocking {
  padding: 0;
}

/* Budget slider card */
.budget-label-row {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-bottom: 0.75rem;
}

.budget-label {
  font-size: 0.875rem;
  font-weight: 600;
  color: #475569;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.budget-value {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0284c7;
  letter-spacing: -0.025em;
}

.budget-slider {
  width: 100%;
  height: 6px;
  border-radius: 3px;
  accent-color: #0284c7;
  cursor: pointer;
}

.slider-range-labels {
  display: flex;
  justify-content: space-between;
  margin-top: 0.375rem;
  font-size: 0.75rem;
  color: #94a3b8;
  font-weight: 500;
}

/* Order action area */
.order-action {
  margin-top: 1.25rem;
  padding-top: 1rem;
  border-top: 1px solid #e2e8f0;
  display: flex;
  justify-content: flex-end;
}

.btn-primary {
  padding: 0.625rem 1.5rem;
  background: #2563eb;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 0.938rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s ease;
}

.btn-primary:hover:not(:disabled) {
  background: #1d4ed8;
}

.btn-primary:disabled {
  background: #94a3b8;
  cursor: not-allowed;
}

/* Success alert */
.order-success {
  background: #d1fae5;
  border: 1px solid #6ee7b7;
  color: #065f46;
  padding: 0.875rem 1.25rem;
  border-radius: 8px;
  margin-bottom: 1.25rem;
  font-size: 0.938rem;
  font-weight: 500;
}

/* Excluded items note */
.excluded-note {
  font-size: 0.875rem;
  color: #64748b;
  margin-bottom: 0.875rem;
  font-style: italic;
}

/* No recommendations placeholder */
.no-recommendations {
  text-align: center;
  padding: 2.5rem;
  color: #64748b;
  font-size: 0.938rem;
}

/* Demand gap cell coloring */
.demand-gap {
  color: #059669;
  font-weight: 600;
}

.demand-gap-neutral {
  color: #64748b;
}

.demand-gap-negative {
  color: #dc2626;
}
</style>
