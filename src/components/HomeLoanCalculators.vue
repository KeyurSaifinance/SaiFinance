<template>
  <section class="bg-gray-100 py-12">
    <div class="max-w-7xl mx-auto px-6">
      <h1 class="mb-10 font-bold text-[30px] lg:text-[30px] leading-[48px] tracking-normal text-primary">
        Home loan repayment calculator
      </h1>

      <div class="grid lg:grid-cols-2 gap-10">
        <!-- LEFT FORM -->
        <div class="bg-white p-8 rounded-lg shadow">
          <label class="text-sm text-gray-600">Property type</label>

          <div class="flex gap-3 mt-2 mb-6 flex-wrap">
            <button
              class="px-6 py-2 rounded border"
              :class="form.propertyType === 'owner_occupier' ? 'bg-primary text-white border-primary' : 'bg-white text-gray-700 border-gray-300'"
              @click="form.propertyType = 'owner_occupier'"
              type="button"
            >
              Owner occupier
            </button>

            <button
              class="px-6 py-2 rounded border"
              :class="form.propertyType === 'investment' ? 'bg-primary text-white border-primary' : 'bg-white text-gray-700 border-gray-300'"
              @click="form.propertyType = 'investment'"
              type="button"
            >
              Residential investment
            </button>
          </div>

          <label class="text-sm text-gray-600">Loan amount</label>

          <input
            v-model.number="form.loanAmount"
            type="number"
            min="0"
            class="w-full border rounded p-3 mt-2 mb-6"
          />

          <label class="text-sm text-gray-600">Variable or fixed rate</label>

          <div class="flex gap-3 mt-2 mb-6 flex-wrap">
            <button
              class="px-6 py-2 rounded border"
              :class="form.rateType === 'Variable' ? 'bg-primary text-white border-primary' : 'bg-white text-gray-700 border-gray-300'"
              @click="setRateType('Variable')"
              type="button"
            >
              Variable
            </button>

            <button
              class="px-6 py-2 rounded border"
              :class="form.rateType === 'Fixed' ? 'bg-primary text-white border-primary' : 'bg-white text-gray-700 border-gray-300'"
              @click="setRateType('Fixed')"
              type="button"
            >
              Fixed
            </button>
          </div>

          <label class="text-sm text-gray-600">Payment type</label>

          <select
            v-model="form.paymentType"
            class="w-full border rounded p-3 mt-2 mb-6"
          >
            <option
              v-for="option in paymentTypeOptions"
              :key="option.value"
              :value="option.value"
            >
              {{ option.label }}
            </option>
          </select>

          <label class="text-sm text-gray-600">Interest rate</label>

          <select
            v-model="form.selectedRateId"
            class="w-full border rounded p-3 mt-2"
          >
            <option value="">Please select</option>
            <option
              v-for="option in availableRateOptions"
              :key="option.id"
              :value="option.id"
            >
              {{ option.label }}
            </option>
          </select>

          <div class="text-sm text-gray-600 mt-2 mb-6">
            Comparison rate:
            <span>{{ selectedComparisonRateText }}</span>
          </div>

          <label class="text-sm text-gray-600">Loan period in years</label>

          <select
            v-model.number="form.loanYears"
            class="w-full border rounded p-3 mt-2 mb-6"
          >
            <option
              v-for="year in yearOptions"
              :key="year"
              :value="year"
            >
              {{ year }}
            </option>
          </select>

          <button
            class="w-full bg-primary text-white py-3 rounded font-semibold"
            @click="calculate"
            type="button"
          >
            Calculate my payments
          </button>
        </div>

        <!-- RIGHT RESULT -->
        <div class="bg-white p-8 rounded-lg shadow">
          <div class="flex justify-between items-center mb-6 gap-4 flex-wrap">
            <h2 class="text-2xl font-semibold text-primary">
              Your estimated repayments
            </h2>

            <select
              v-model="form.frequency"
              class="border rounded px-3 py-2"
              @change="recalculateIfPossible"
            >
              <option value="Monthly">Monthly</option>
              <option value="Fortnightly">Fortnightly</option>
              <option value="Weekly">Weekly</option>
            </select>
          </div>

          <h3 class="text-5xl font-bold text-primary mb-8">
            {{ formatCurrency(result.amount) }}
          </h3>

          <div
            v-if="result.ioMessage"
            class="text-gray-700 mb-6"
          >
            {{ result.ioMessage }}
          </div>

          <div class="space-y-4 text-gray-600">
            <div class="flex justify-between border-b pb-2 gap-4">
              <span>Interest rate</span>
              <span class="text-right">{{ result.showRate }}</span>
            </div>

            <div class="flex justify-between border-b pb-2 gap-4">
              <span>Comparison rate</span>
              <span class="text-right">{{ result.showComparisonRate }}</span>
            </div>

            <div class="flex justify-between border-b pb-2 gap-4">
              <span>Interest rate type</span>
              <span class="text-right">{{ form.rateType }}</span>
            </div>

            <div class="flex justify-between border-b pb-2 gap-4">
              <span>Payment type</span>
              <span class="text-right">{{ selectedPaymentTypeLabel }}</span>
            </div>

            <div class="flex justify-between border-b pb-2 gap-4">
              <span>Loan period</span>
              <span class="text-right">{{ form.loanYears }} years</span>
            </div>

            <div class="flex justify-between border-b pb-2 gap-4">
              <span>Property type</span>
              <span class="text-right">
                {{ form.propertyType === 'owner_occupier' ? 'Owner occupier' : 'Residential investment' }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { computed, reactive, watch } from 'vue'

const rateOptions = {
  variable: {
    pi: [
      { value: '6.74', label: '6.74% p.a standard variable 80% or less LVR' },
      { value: '6.94', label: '6.94% p.a standard variable more than 80% LVR' },
      { value: '7.49', label: '7.49% p.a simplicity PLUS' },
      { value: '5.89', label: '5.89% p.a simplicity PLUS special offer discount 60% or less LVR*' },
      { value: '5.94', label: '5.94% p.a simplicity PLUS special offer discount 70% or less LVR*' },
      { value: '6.04', label: '6.04% p.a simplicity PLUS special offer discount 80% or less LVR*' },
      { value: '6.59', label: '6.59% p.a simplicity PLUS special offer discount 90% or less LVR*' },
      { value: '7.14', label: '7.14% p.a simplicity PLUS special offer discount more than 90% LVR*' },
    ],
    io: [
      { value: '7.29', label: '7.29% p.a standard variable 80% or less LVR', comparison: '6.98' },
      { value: '7.49', label: '7.49% p.a standard variable more than 80% LVR', comparison: '7.19' },
    ],
  },
  fixed: {
    pi: {
      '1': [
        { value: '6.79', label: '6.79% p.a 1 year fixed' },
        { value: '6.34', label: '6.34% p.a 1 year fixed 80% or less LVR' },
      ],
      '2': [
        { value: '6.64', label: '6.64% p.a 2 year fixed' },
        { value: '6.19', label: '6.19% p.a 2 year fixed 80% or less LVR' },
      ],
      '3': [
        { value: '6.94', label: '6.94% p.a 3 year fixed' },
        { value: '6.49', label: '6.49% p.a 3 year fixed 80% or less LVR' },
      ],
      '4': [
        { value: '6.99', label: '6.99% p.a 4 year fixed' },
        { value: '6.54', label: '6.54% p.a 4 year fixed 80% or less LVR' },
      ],
      '5': [
        { value: '7.09', label: '7.09% p.a 5 year fixed' },
        { value: '6.64', label: '6.64% p.a 5 year fixed 80% or less LVR' },
      ],
    },
    piAll: [
      { value: '6.34', label: '6.34% p.a 1 year fixed' },
      { value: '5.89', label: '5.89% p.a 1 year fixed 80% or less LVR' },
      { value: '6.24', label: '6.24% p.a 2 year fixed' },
      { value: '5.79', label: '5.79% p.a 2 year fixed 80% or less LVR' },
      { value: '6.49', label: '6.49% p.a 3 year fixed' },
      { value: '6.04', label: '6.04% p.a 3 year fixed 80% or less LVR' },
      { value: '6.54', label: '6.54% p.a 4 year fixed' },
      { value: '6.69', label: '6.69% p.a 5 year fixed 80% or less LVR' },
      { value: '6.24', label: '6.24% p.a 5 year fixed' },
      { value: '5.89', label: '5.89% p.a 1 year fixed 80% or less LVR' },
    ],
  },
}

const form = reactive({
  propertyType: 'owner_occupier',
  loanAmount: 500000,
  rateType: 'Variable',
  paymentType: 'pi',
  selectedRateId: '',
  loanYears: 30,
  frequency: 'Monthly',
})

const result = reactive({
  amount: 0,
  showRate: '-%',
  showComparisonRate: '-%',
  ioMessage: '',
})

const yearOptions = Array.from({ length: 30 }, (_, index) => index + 1)

const paymentTypeOptions = computed(() => {
  if (form.rateType === 'Fixed') {
    return [
      { value: 'pi', label: 'Principal and Interest' },
      { value: 'fixed-1', label: '1 year fixed' },
      { value: 'fixed-2', label: '2 year fixed' },
      { value: 'fixed-3', label: '3 year fixed' },
      { value: 'fixed-4', label: '4 year fixed' },
      { value: 'fixed-5', label: '5 year fixed' },
    ]
  }

  return [
    { value: 'pi', label: 'Principal and Interest' },
    { value: 'io-1', label: '1 year interest only' },
    { value: 'io-2', label: '2 year interest only' },
    { value: 'io-3', label: '3 year interest only' },
    { value: 'io-4', label: '4 year interest only' },
    { value: 'io-5', label: '5 year interest only' },
  ]
})

const availableRateOptions = computed(() => {
  const typeKey = form.rateType === 'Fixed' ? 'fixed' : 'variable'
  let options = []

  if (typeKey === 'fixed') {
    if (form.paymentType === 'pi') {
      options = rateOptions.fixed.piAll
    } else {
      const termKey = form.paymentType.split('-')[1] || '1'
      options = rateOptions.fixed.pi[termKey] || []
    }
  } else {
    const key = form.paymentType === 'pi' ? 'pi' : 'io'
    options = rateOptions.variable[key] || []
  }

  return options.map((option, index) => ({
    ...option,
    id: `${typeKey}-${form.paymentType}-${index}`,
  }))
})

const selectedRateOption = computed(() => {
  return availableRateOptions.value.find(
    (option) => option.id === form.selectedRateId
  ) || null
})

const selectedComparisonRateText = computed(() => {
  const option = selectedRateOption.value
  if (!option) return '-%'
  return `${option.comparison || option.value}% p.a`
})

const selectedPaymentTypeLabel = computed(() => {
  return paymentTypeOptions.value.find(
    (option) => option.value === form.paymentType
  )?.label || '-'
})

watch(
  () => form.rateType,
  () => {
    form.paymentType = 'pi'
    form.selectedRateId = ''
    result.amount = 0
    result.showRate = '-%'
    result.showComparisonRate = '-%'
    result.ioMessage = ''
  }
)

watch(
  () => form.paymentType,
  () => {
    form.selectedRateId = ''
    result.amount = 0
    result.showRate = '-%'
    result.showComparisonRate = '-%'
    result.ioMessage = ''
  }
)

function setRateType(type) {
  form.rateType = type
}

function getFrequencyMultiplier() {
  if (form.frequency === 'Weekly') return 12 / 52
  if (form.frequency === 'Fortnightly') return 12 / 26
  return 1
}

function formatFrequencyLabel() {
  if (form.frequency === 'Weekly') return 'per week'
  if (form.frequency === 'Fortnightly') return 'per fortnight'
  return 'per month'
}

function calculate() {
  const loan = Number(form.loanAmount || 0)
  const selectedOption = selectedRateOption.value
  const annualRate = Number(selectedOption?.value || 0)
  const years = Number(form.loanYears || 0)

  if (!loan || !annualRate || !years || !selectedOption) {
    result.amount = 0
    result.showRate = '-%'
    result.showComparisonRate = '-%'
    result.ioMessage = ''
    return
  }

  const monthlyRate = annualRate / 100 / 12
  const months = years * 12

  let monthlyPayment = 0

  if (form.paymentType.startsWith('io')) {
    monthlyPayment = loan * monthlyRate
  } else {
    monthlyPayment =
      (loan * monthlyRate * Math.pow(1 + monthlyRate, months)) /
      (Math.pow(1 + monthlyRate, months) - 1)
  }

  const displayPayment = monthlyPayment * getFrequencyMultiplier()

  result.amount = Number.isFinite(displayPayment) ? displayPayment : 0
  result.showRate = selectedOption.label
  result.showComparisonRate = `${selectedOption.comparison || selectedOption.value}% p.a`
  result.ioMessage = ''

  if (form.paymentType.startsWith('io')) {
    const ioYears = parseInt(form.paymentType.split('-')[1], 10)
    const remainingYears = years - ioYears

    if (remainingYears > 0) {
      const remainingMonths = remainingYears * 12
      const remainingPayment =
        (loan * monthlyRate * Math.pow(1 + monthlyRate, remainingMonths)) /
        (Math.pow(1 + monthlyRate, remainingMonths) - 1)

      if (Number.isFinite(remainingPayment)) {
        const remainingDisplay = remainingPayment * getFrequencyMultiplier()
        result.ioMessage =
          `After the ${ioYears} year interest only period ends, based on current rates (which may change) your estimated payments will be ${formatCurrency(remainingDisplay)} ${formatFrequencyLabel()} over the remaining loan term.`
      }
    }
  }
}

function recalculateIfPossible() {
  if (result.showRate !== '-%') {
    calculate()
  }
}

function formatCurrency(value) {
  return new Intl.NumberFormat('en-AU', {
    style: 'currency',
    currency: 'AUD',
    maximumFractionDigits: 0,
  }).format(Math.max(0, Number(value || 0)))
}
</script>