<template>
  <div class="analysis-page h-full overflow-y-auto">
    <div class="mx-auto w-full max-w-5xl px-6 py-8">
      <!-- Header -->
      <div class="mb-8 flex items-start justify-between">
        <div>
          <h2 class="text-2xl-semibold text-ink-gray-9">
            {{ __('Analysis') }}
          </h2>

          <p class="mt-1 text-base text-ink-gray-5">
            {{ __('Evaluate the funding opportunity and determine its priority.') }}
          </p>
        </div>

        <div class="score-summary">
          <div class="score-label">
            {{ __('Overall Score') }}
          </div>

          <div class="score-value">
            {{ overallScore }}
            <span>/ 40</span>
          </div>

          <div
            class="priority-value"
            :class="priorityClass"
          >
            {{ priority || __('Not evaluated') }}
          </div>
        </div>
      </div>

      <!-- Scoring -->
      <div class="analysis-card">
        <div class="card-header">
          <div>
            <h3>{{ __('Scoring') }}</h3>
            <p>
              {{ __('Rate each criterion from 1 to 5.') }}
            </p>
          </div>
        </div>

        <div class="grid grid-cols-1 gap-5 md:grid-cols-2">
          <div
            v-for="field in scoreFields"
            :key="field.name"
            class="field-wrapper"
          >
            <label class="field-label">
              {{ __(field.label) }}
            </label>

            <Select
              class="w-full"
              :modelValue="doc[field.name] || ''"
              :options="scoreOptions"
              :placeholder="__('Select score')"
              @update:modelValue="(value) => saveScore(field.name, value)"
            />
          </div>
        </div>
      </div>

      <!-- Score Result -->
      <div class="analysis-card mt-6">
        <div class="card-header">
          <div>
            <h3>{{ __('Assessment Result') }}</h3>
            <p>
              {{ __('Overall Score and Priority are calculated automatically.') }}
            </p>
          </div>
        </div>

        <div class="grid grid-cols-1 gap-5 md:grid-cols-2">
          <div class="field-wrapper">
            <label class="field-label">
              {{ __('Overall Score') }}
            </label>

            <div class="readonly-field">
              {{ overallScore }} / 40
            </div>
          </div>

          <div class="field-wrapper">
            <label class="field-label">
              {{ __('Priority') }}
            </label>

            <div
              class="readonly-field priority-field"
              :class="priorityClass"
            >
              {{ priority || __('Not evaluated') }}
            </div>
          </div>
        </div>
      </div>

      <!-- Recommendation -->
      <div class="analysis-card mt-6">
        <div class="card-header">
          <div>
            <h3>{{ __('Recommendation & Action') }}</h3>
          </div>
        </div>

        <div class="grid grid-cols-1 gap-5 md:grid-cols-2">
          <div class="field-wrapper md:col-span-2">
            <label class="field-label">
              {{ __('Recommendation') }}
            </label>

            <Textarea
              class="w-full"
              :modelValue="doc.recommendation || ''"
              :rows="3"
              placeholder="Enter recommendation..."
              @change="saveField('recommendation', $event.target.value)"
            />
          </div>

          <div class="field-wrapper">
            <label class="field-label">
              {{ __('Lead Person') }}
            </label>

            <Link
              class="w-full"
              doctype="Employee"
              :value="doc.lead_person || ''"
              :placeholder="__('Select Lead Person...')"
              @change="(value) => saveField('lead_person', value)"
            />
          </div>

          <div class="field-wrapper">
            <label class="field-label">
              {{ __('Decision Deadline') }}
            </label>

            <DatePicker
              class="w-full"
              :value="doc.decision_deadline || ''"
              :placeholder="__('Select date...')"
              @change="(value) => saveField('decision_deadline', value)"
            />
          </div>

          <div class="field-wrapper md:col-span-2">
            <label class="field-label">
              {{ __('Next Action') }}
            </label>

            <Textarea
              class="w-full"
              :modelValue="doc.next_action || ''"
              :rows="3"
              placeholder="Enter next action..."
              @change="saveField('next_action', $event.target.value)"
            />
          </div>
        </div>
      </div>

      <!-- Status / Risk -->
      <div class="analysis-card mt-6">
        <div class="card-header">
          <div>
            <h3>{{ __('Status & Risk') }}</h3>
          </div>
        </div>

        <div class="grid grid-cols-1 gap-5 md:grid-cols-2">
          <div class="field-wrapper">
            <label class="field-label">
              {{ __('Status') }}
            </label>

            <Select
              class="w-full"
              :modelValue="doc.analysis_status || ''"
              :options="statusOptions"
              :placeholder="__('Select status')"
              @update:modelValue="
                (value) => saveField('analysis_status', value)
              "
            />
          </div>

          <div class="field-wrapper">
            <label class="field-label">
              {{ __('Competition / Risk') }}
            </label>

            <Textarea
              class="w-full"
              :modelValue="doc.competition_risk || ''"
              :rows="3"
              placeholder="Enter competition or risk..."
              @change="saveField('competition_risk', $event.target.value)"
            />
          </div>

          <div class="field-wrapper md:col-span-2">
            <label class="field-label">
              {{ __('Key Rationale') }}
            </label>

            <Textarea
              class="w-full"
              :modelValue="doc.key_rationale || ''"
              :rows="4"
              placeholder="Enter key rationale..."
              @change="saveField('key_rationale', $event.target.value)"
            />
          </div>

          <div class="field-wrapper md:col-span-2">
            <label class="field-label">
              {{ __('Source / Link') }}
            </label>

            <TextInput
              class="w-full"
              type="url"
              :modelValue="doc.analysis_source_link || ''"
              placeholder="https://..."
              @change="
                saveField('analysis_source_link', $event.target.value)
              "
            />

            <a
              v-if="isUrl(doc.analysis_source_link)"
              :href="doc.analysis_source_link"
              target="_blank"
              rel="noopener noreferrer"
              class="mt-2 inline-block text-sm text-ink-blue-5 hover:underline"
            >
              {{ __('Open Source Link') }}
            </a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import Link from '@/components/Controls/Link.vue'
import {
  DatePicker,
  Select,
  Textarea,
  TextInput,
  toast,
} from 'frappe-ui'
import { computed } from 'vue'

const props = defineProps({
  document: {
    type: Object,
    required: true,
  },
})

const document = props.document
const doc = computed(() => document.doc || {})

const scoreFields = [
  {
    name: 'strategic_fit',
    label: 'Strategic Fit',
  },
  {
    name: 'eligibility',
    label: 'Eligibility',
  },
  {
    name: 'programmatic_fit',
    label: 'Programmatic Fit',
  },
  {
    name: 'geographic_fit',
    label: 'Geographic Fit',
  },
  {
    name: 'partnership_modality',
    label: 'Partnership Modality',
  },
  {
    name: 'funding_budget_fit',
    label: 'Funding / Budget Fit',
  },
  {
    name: 'yldf_capacity',
    label: 'YLDF Capacity',
  },
  {
    name: 'deadline_feasibility',
    label: 'Deadline Feasibility',
  },
]

const scoreOptions = ['1', '2', '3', '4', '5']

const statusOptions = [
  'New',
  'Eligibility check',
  'Urgent / conditional',
  'Time-critical',
  'Next cycle',
  'Paused / monitor',
  'In development',
  'Submitted',
  'Closed',
]

const overallScore = computed(() => {
  return scoreFields.reduce((total, field) => {
    return total + Number(doc.value[field.name] || 0)
  }, 0)
})

const priority = computed(() => {
  const score = overallScore.value

  if (score >= 32) {
    return 'High priority'
  }

  if (score >= 24) {
    return 'Potential / discuss'
  }

  return 'Low priority'
})

const priorityClass = computed(() => {
  if (overallScore.value >= 32) {
    return 'priority-high'
  }

  if (overallScore.value >= 24) {
    return 'priority-potential'
  }

  return 'priority-low'
})

function saveScore(fieldname, value) {
  saveField(fieldname, value)
}

function saveField(fieldname, value) {
  const oldValue = doc.value[fieldname]

  doc.value[fieldname] = value

  // Update calculated values immediately in the UI.
  doc.value.overall_score = overallScore.value
  doc.value.priority = priority.value

  document.save.submit(null, {
    onSuccess: () => {
      // Server-side validation also calculates these values.
    },

    onError: () => {
      doc.value[fieldname] = oldValue

      // Recalculate after rollback.
      doc.value.overall_score = overallScore.value
      doc.value.priority = priority.value

      toast.error(__('Could not save the Analysis field.'))
    },
  })
}

function isUrl(value) {
  return typeof value === 'string' && /^https?:\/\//i.test(value.trim())
}
</script>

<style scoped>
.analysis-page {
  background: var(--surface-white);
}

.analysis-card {
  border: 1px solid var(--border-color);
  border-radius: 10px;
  background: var(--surface-white);
  padding: 24px;
}

.card-header {
  margin-bottom: 20px;
}

.card-header h3 {
  font-size: 16px;
  font-weight: 600;
  color: var(--ink-gray-9);
}

.card-header p {
  margin-top: 4px;
  font-size: 13px;
  color: var(--ink-gray-5);
}

.field-wrapper {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.field-label {
  font-size: 13px;
  font-weight: 500;
  color: var(--ink-gray-7);
}

.readonly-field {
  display: flex;
  min-height: 40px;
  align-items: center;
  border: 1px solid var(--border-color);
  border-radius: 7px;
  padding: 8px 12px;
  background: var(--surface-gray-1);
  font-size: 15px;
  font-weight: 600;
  color: var(--ink-gray-9);
}

.score-summary {
  min-width: 150px;
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 12px 16px;
  text-align: center;
}

.score-label {
  font-size: 12px;
  color: var(--ink-gray-5);
}

.score-value {
  margin-top: 2px;
  font-size: 24px;
  font-weight: 700;
  color: var(--ink-gray-9);
}

.score-value span {
  font-size: 13px;
  font-weight: 400;
  color: var(--ink-gray-5);
}

.priority-value {
  margin-top: 4px;
  font-size: 12px;
  font-weight: 600;
}

.priority-high {
  color: var(--ink-green-6);
}

.priority-potential {
  color: var(--ink-orange-6);
}

.priority-low {
  color: var(--ink-gray-6);
}

.priority-field {
  background: var(--surface-gray-1);
}

:deep(input),
:deep(textarea),
:deep(button) {
  border-radius: 7px;
}

:deep(textarea) {
  min-height: 80px;
}
</style>
