# Frontend Code Style Guide (Vue 3 + Vite + Vuetify)

## Table of Contents
1. [Project Structure](#project-structure)
2. [Vue 3 Composition API](#vue-3-composition-api)
3. [Component Guidelines](#component-guidelines)
4. [Script Setup](#script-setup)
5. [Template Guidelines](#template-guidelines)
6. [Style Guidelines](#style-guidelines)
7. [API Integration](#api-integration)
8. [Naming Conventions](#naming-conventions)
9. [Best Practices](#best-practices)

## Project Structure

```
frontend/
├── src/
│   ├── components/          # Reusable Vue components
│   │   ├── ContactList.vue
│   │   └── ContactForm.vue
│   ├── services/            # API and business logic
│   │   ├── api.js          # Axios instance and interceptors
│   │   └── contactService.js # Contact API calls
│   ├── assets/             # Static assets
│   ├── App.vue             # Root component
│   └── main.js             # Application entry point
├── index.html
├── vite.config.js
└── package.json
```

## Vue 3 Composition API

### Preferred Approach
Use Vue 3 Composition API (`<script setup>`) for all new components.

```vue
<script setup>
import { ref, computed, onMounted } from 'vue'

// Reactive state
const count = ref(0)

// Computed values
const doubleCount = computed(() => count.value * 2)

// Lifecycle hooks
onMounted(() => {
  console.log('Component mounted')
})
</script>
```

### Reactive References
- Use `ref()` for primitive values and objects
- Use `reactive()` for objects with multiple properties
- Use `computed()` for derived state

```vue
<script setup>
// ✓ Good
const user = ref({
  name: '',
  email: ''
})

// ✓ Good
const isValid = computed(() => user.value.name.length > 0)
</script>
```

## Component Guidelines

### Single Responsibility
Each component should have a single, well-defined purpose.

```vue
<!-- ✓ Good: ContactList focuses on listing contacts -->
<template>
  <div class="contact-list">
    <!-- Implementation -->
  </div>
</template>

<script setup>
// Component logic
</script>
```

### Component Communication

**Props (Parent → Child)**
```vue
<script setup>
const props = defineProps({
  contact: {
    type: Object,
    default: () => ({})
  },
  isEditing: {
    type: Boolean,
    default: false
  }
})
</script>
```

**Emits (Child → Parent)**
```vue
<script setup>
const emit = defineEmits(['submit', 'cancel'])

// Emit events
const handleSubmit = () => {
  emit('submit', formData)
}
</script>
```

### Props and Emits Naming
- Props: camelCase in JavaScript, kebab-case in template
- Emits: kebab-case (recommended)

```vue
<!-- Template -->
<ContactForm
  :contact-data="contact"
  @form-submitted="handleSubmit"
/>

<!-- Script -->
<script setup>
const props = defineProps({
  contactData: Object
})

const emit = defineEmits(['form-submitted'])
</script>
```

## Script Setup

### Script Setup Syntax
Use `<script setup>` for all Vue 3 components.

```vue
<script setup>
import { ref } from 'vue'

const message = ref('Hello World')
</script>

<template>
  <div>{{ message }}</div>
</template>
```

### Top-Level Constants and Functions
Place constants and helper functions at the top level.

```vue
<script setup>
import { ref, computed } from 'vue'

// ✓ Good: Top-level constants
const API_BASE_URL = '/api'

// ✓ Good: Top-level functions
const formatDate = (date) => {
  return new Date(date).toLocaleDateString()
}

// Component logic
const currentDate = ref(new Date())
</script>
```

## Template Guidelines

### Directive Ordering
When using multiple directives on an element, follow this order:

1. `v-if`, `v-else-if`, `v-else`
2. `v-for`
3. `v-once`
4. `v-show`
5. `v-bind` (`:`)
6. `v-model`
7. `v-on` (`@`)
8. `v-html` / `v-text`

```vue
<!-- ✓ Good -->
<li
  v-if="isVisible"
  v-for="item in items"
  :key="item.id"
  :class="{ active: item.isActive }"
  @click="handleClick"
>
  {{ item.name }}
</li>
```

### Template Expressions
Keep template expressions simple and readable.

```vue
<!-- ✓ Good -->
<template>
  <div>
    <h1>{{ pageTitle }}</h1>
    <span>{{ fullName }}</span>
  </div>
</template>

<script setup>
const pageTitle = 'Contact List'
const fullName = computed(() => `${firstName} ${lastName}`)
</script>
```

## Style Guidelines

### Scoped Styles
Always use `scoped` attribute for component styles.

```vue
<style scoped>
.contact-list {
  padding: 20px;
}

.contact-list .header {
  margin-bottom: 20px;
}
</style>
```

### CSS Naming Conventions
- Use kebab-case for CSS classes
- Use BEM methodology for complex components

```vue
<style scoped>
/* BEM example */
.contact-form {
  /* Block */
}

.contact-form__field {
  /* Element */
}

.contact-form__field--error {
  /* Modifier */
}
</style>
```

### Vuetify 3 Integration
Follow Vuetify 3 component conventions:

```vue
<template>
  <!-- ✓ Good: Use Vuetify components -->
  <v-card>
    <v-card-title>Contact Details</v-card-title>
    <v-card-text>
      <v-text-field
        v-model="form.name"
        label="Name"
        :rules="[rules.required]"
      />
    </v-card-text>
  </v-card>
</template>

<script setup>
const rules = {
  required: value => !!value || 'Required'
}
</script>
```

## API Integration

### Service Layer Pattern
Use dedicated service files for API calls.

```javascript
// services/contactService.js
import apiClient from './api'

export const contactService = {
  /**
   * Fetch all contacts
   * @returns {Promise} Promise resolving to API response
   */
  async getContacts() {
    return apiClient.get('/contacts')
  },

  /**
   * Create a new contact
   * @param {Object} contactData - Contact information
   * @returns {Promise} Promise resolving to API response
   */
  async createContact(contactData) {
    return apiClient.post('/contacts', contactData)
  }
}
```

### Axios Configuration
Centralize API configuration in `api.js`.

```javascript
// services/api.js
import axios from 'axios'

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
})

// Response interceptor
apiClient.interceptors.response.use(
  response => response.data,
  error => Promise.reject(error)
)

export default apiClient
```

## Naming Conventions

### Files and Folders
- **Components**: PascalCase (e.g., `ContactList.vue`, `UserProfile.vue`)
- **Services**: camelCase (e.g., `contactService.js`, `userService.js`)
- **Utilities**: camelCase (e.g., `helpers.js`, `formatters.js`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `API_ENDPOINTS.js`)

### Variables and Functions
- Use camelCase
- Use meaningful, descriptive names

```javascript
// ✓ Good
const userProfile = ref({})
const isLoading = ref(false)

const fetchUserProfile = async () => { }

// ✗ Avoid
const user = ref({})
const loading = ref(false)

const getData = async () => { }
```

### Constants
Use UPPER_SNAKE_CASE with meaningful names.

```javascript
// ✓ Good
const MAX_RETRY_COUNT = 3
const API_TIMEOUT = 10000
const DEFAULT_PAGE_SIZE = 20
```

## Best Practices

### 1. Avoid Side Effects in Computed Properties
```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)

// ✓ Good
const doubleCount = computed(() => count.value * 2)

// ✗ Avoid
const badComputed = computed(() => {
  console.log('This is a side effect!')
  return count.value * 2
})
</script>
```

### 2. Use Key with v-for
```vue
<template>
  <!-- ✓ Good -->
  <li v-for="item in items" :key="item.id">
    {{ item.name }}
  </li>
</template>
```

### 3. Optimize Reactivity
```vue
<script setup>
import { ref, shallowRef } from 'vue'

// For large objects that don't need deep reactivity
const largeData = shallowRef([])

// For primitives and small objects
const count = ref(0)
const user = ref({ name: '' })
</script>
```

### 4. Error Handling
```vue
<script setup>
import { ref } from 'vue'

const error = ref(null)
const loading = ref(false)

const fetchData = async () => {
  loading.value = true
  error.value = null

  try {
    // API call
  } catch (err) {
    error.value = err.message || 'An error occurred'
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div>
    <v-alert v-if="error" type="error">
      {{ error }}
    </v-alert>
    <v-progress-linear v-if="loading" indeterminate />
  </div>
</template>
```

### 5. Cleanup in onUnmounted
```vue
<script setup>
import { onMounted, onUnmounted } from 'vue'

let intervalId = null

onMounted(() => {
  intervalId = setInterval(() => {
    // Do something
  }, 1000)
})

onUnmounted(() => {
  if (intervalId) {
    clearInterval(intervalId)
  }
})
</script>
```

### 6. Prop Validation
```vue
<script setup>
const props = defineProps({
  // Required string
  title: {
    type: String,
    required: true
  },

  // Optional with default
  size: {
    type: String,
    default: 'medium',
    validator: (value) => ['small', 'medium', 'large'].includes(value)
  },

  // Object with specific shape
  user: {
    type: Object,
    default: () => ({})
  }
})
</script>
```

## Code Review Checklist

- [ ] Component follows single responsibility principle
- [ ] Uses `<script setup>` and Composition API
- [ ] Proper reactive references (ref vs reactive)
- [ ] All props have type definitions
- [ ] All emits are defined with `defineEmits`
- [ ] Template expressions are simple and readable
- [ ] Styles are scoped
- [ ] Proper error handling
- [ ] JSDoc comments for complex functions
- [ ] Follows naming conventions
- [ ] No console.log in production code
- [ ] v-for has :key attribute
- [ ] Computed properties have no side effects
- [ ] Cleanup in onUnmounted if needed

## Resources

- [Vue 3 Style Guide](https://vuejs.org/style-guide/)
- [Vue 3 Composition API](https://vuejs.org/guide/extras/composition-api-faq.html)
- [Vuetify 3 Documentation](https://vuetifyjs.com/)
- [Vite Documentation](https://vitejs.dev/)
