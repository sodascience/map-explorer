<template>
  <div class="max-w-md mx-auto px-4 py-6">
    <!-- Header with progress indicator -->
    <div class="mb-6">
      <h2 class="text-xl font-bold text-gray-800">{{ steps[currentStep].title }}</h2>
      <div class="flex items-center mt-4">
        <div v-for="(step, index) in steps" :key="index" class="flex items-center">
          <div
            class="w-8 h-8 rounded-full flex items-center justify-center text-sm"
            :class="{
              'bg-blue-500 text-white': index < currentStep,
              'bg-blue-600 text-white font-bold': index === currentStep,
              'bg-gray-200 text-gray-600': index > currentStep
            }"
          >
            {{ index + 1 }}
          </div>
          <div v-if="index < steps.length - 1" class="w-10 h-1 bg-gray-200 mx-1"
               :class="{ 'bg-blue-500': index < currentStep }">
          </div>
        </div>
      </div>
    </div>

    <!-- Step content -->
    <div class="bg-white rounded-lg shadow-sm p-6 mb-6 border border-gray-200">
      <!-- File selection -->
      <div v-if="currentStep === 0">
        <label class="block text-gray-700 mb-2">Select a file to import:</label>
        <div class="border-2 border-dashed border-gray-300 rounded-lg p-6 text-center hover:border-blue-500 transition-colors">
          <input
            type="file"
            @change="handleFileSelect"
            class="hidden"
            id="fileInput"
            accept=".csv,.xlsx,.json"
          />
          <label for="fileInput" class="cursor-pointer">
            <div class="flex flex-col items-center">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-10 w-10 text-gray-400 mb-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12" />
              </svg>
              <span class="text-gray-600">Click to select a file or drag and drop</span>
              <span class="text-sm text-gray-500 mt-1">Supported formats: CSV, XLSX, JSON</span>
            </div>
          </label>
          <div v-if="formData.file" class="mt-4 text-left">
            <div class="flex items-center">
              <span class="text-green-600">✓</span>
              <span class="ml-2 text-gray-700">{{ formData.file.name }}</span>
              <button
                @click.prevent="removeFile"
                class="ml-auto text-red-500 hover:text-red-700"
              >
                Remove
              </button>
            </div>
          </div>
        </div>
        <p v-if="errors.file" class="text-red-500 text-sm mt-2">{{ errors.file }}</p>
      </div>

      <!-- Mapping fields -->
      <div v-if="currentStep === 1">
        <label class="block text-gray-700 mb-3">Map the fields:</label>
        <div v-for="(field, index) in importFields" :key="index" class="mb-4">
          <label class="block text-gray-600 text-sm mb-1">{{ field.label }}:</label>
          <select
            v-model="formData.mappings[field.key]"
            class="w-full p-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option value="">-- Select field --</option>
            <option v-for="option in fileFields" :key="option" :value="option">
              {{ option }}
            </option>
          </select>
          <p v-if="errors.mappings && errors.mappings[field.key]" class="text-red-500 text-sm mt-1">
            {{ errors.mappings[field.key] }}
          </p>
        </div>
      </div>

      <!-- Data preview -->
      <div v-if="currentStep === 2">
        <label class="block text-gray-700 mb-3">Verify your data:</label>
        <div class="overflow-x-auto">
          <table class="min-w-full border border-gray-200">
            <thead>
              <tr class="bg-gray-50">
                <th v-for="field in importFields" :key="field.key" class="py-2 px-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider border-b">
                  {{ field.label }}
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(row, rowIndex) in previewData" :key="rowIndex" class="border-b">
                <td v-for="field in importFields" :key="field.key" class="py-2 px-3 text-sm text-gray-700">
                  {{ row[field.key] }}
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <div v-if="previewData.length === 0" class="py-4 text-center text-gray-500">
          No data available for preview
        </div>
      </div>

      <!-- Confirmation -->
      <div v-if="currentStep === 3">
        <div class="text-center py-4">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 mx-auto text-green-500 mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
          <h3 class="text-lg font-semibold mb-2">Ready to import!</h3>
          <p class="text-gray-600 mb-4">
            You're about to import {{ previewData.length }} records.
          </p>
          <div class="bg-gray-50 p-3 rounded-md text-left">
            <p class="text-sm text-gray-600">File: <span class="font-medium">{{ formData.file?.name }}</span></p>
            <p class="text-sm text-gray-600 mt-1">Field mappings:</p>
            <ul class="text-xs text-gray-500 mt-1 ml-4">
              <li v-for="field in importFields" :key="field.key">
                {{ field.label }}: {{ formData.mappings[field.key] }}
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>

    <!-- Navigation -->
    <div class="flex justify-between">
      <button
        v-if="currentStep > 0"
        @click="prevStep"
        class="px-4 py-2 text-gray-600 hover:text-gray-900 flex items-center focus:outline-none"
      >
        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
        </svg>
        Back
      </button>
      <div v-else></div>

      <button
        v-if="currentStep < steps.length - 1"
        @click="nextStep"
        class="px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50"
      >
        {{ steps[currentStep].nextButtonText || 'Continue' }}
      </button>
      <button
        v-else
        @click="finishImport"
        class="px-4 py-2 bg-green-500 text-white rounded-md hover:bg-green-600 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-opacity-50"
      >
        Import Data
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DataImportWizard',
  data() {
    return {
      currentStep: 0,
      steps: [
        {
          title: 'Select File',
          nextButtonText: 'Continue'
        },
        {
          title: 'Map Fields',
          nextButtonText: 'Preview Data'
        },
        {
          title: 'Preview Data',
          nextButtonText: 'Confirm'
        },
        {
          title: 'Confirmation'
        }
      ],
      formData: {
        file: null,
        mappings: {}
      },
      errors: {},
      importFields: [
        { key: 'name', label: 'Name', required: true },
        { key: 'email', label: 'Email', required: true },
        { key: 'phone', label: 'Phone Number', required: false },
        { key: 'status', label: 'Status', required: false }
      ],
      fileFields: [], // Will be populated after file selection
      previewData: []
    }
  },
  methods: {
    handleFileSelect(event) {
      const file = event.target.files[0]
      if (!file) return

      // Validate file type
      const allowedTypes = [
        'text/csv',
        'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
        'application/json'
      ]

      if (!allowedTypes.includes(file.type)) {
        this.errors.file = 'Please select a valid CSV, XLSX, or JSON file'
        return
      }

      this.formData.file = file
      this.errors.file = null

      // In a real application, you would parse the file here
      // For this example, we'll simulate file fields
      this.fileFields = ['first_name', 'last_name', 'user_email', 'contact_number', 'user_status']
    },

    removeFile() {
      this.formData.file = null
      document.getElementById('fileInput').value = ''
    },

    validateStep() {
      this.errors = {}

      if (this.currentStep === 0) {
        if (!this.formData.file) {
          this.errors.file = 'Please select a file to import'
          return false
        }
      }

      if (this.currentStep === 1) {
        const mappingErrors = {}
        let hasError = false

        this.importFields.forEach(field => {
          if (field.required && !this.formData.mappings[field.key]) {
            mappingErrors[field.key] = `${field.label} mapping is required`
            hasError = true
          }
        })

        if (hasError) {
          this.errors.mappings = mappingErrors
          return false
        }
      }

      return true
    },

    prevStep() {
      if (this.currentStep > 0) {
        this.currentStep--
      }
    },

    nextStep() {
      if (this.validateStep()) {
        if (this.currentStep === 1) {
          // Generate preview data when moving to preview step
          this.generatePreviewData()
        }

        if (this.currentStep < this.steps.length - 1) {
          this.currentStep++
        }
      }
    },

    generatePreviewData() {
      // In a real application, this would use the actual file data
      // For this example, we'll generate mock data
      this.previewData = [
        {
          name: 'John Doe',
          email: 'john@example.com',
          phone: '555-123-4567',
          status: 'Active'
        },
        {
          name: 'Jane Smith',
          email: 'jane@example.com',
          phone: '555-987-6543',
          status: 'Pending'
        },
        {
          name: 'Alex Johnson',
          email: 'alex@example.com',
          phone: '555-555-5555',
          status: 'Inactive'
        }
      ]
    },

    finishImport() {
      // In a real application, this would process the import
      // Here we'll just emit an event with the data
      this.$emit('import-complete', {
        file: this.formData.file,
        mappings: this.formData.mappings,
        records: this.previewData
      })

      // You could also reset the wizard here if needed
      // this.resetWizard()
    },

    resetWizard() {
      this.currentStep = 0
      this.formData = {
        file: null,
        mappings: {}
      }
      this.errors = {}
      this.fileFields = []
      this.previewData = []
    }
  }
}
</script>
