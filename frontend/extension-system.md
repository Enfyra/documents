# Extension System

The Extension System allows you to create custom pages and widgets using Vue.js components. Extensions are tightly coupled with the menu system - when you create a menu item, you need an extension to provide the actual content that displays when users click that menu.

**→ [Menu Management Guide](./menu-management.md)** - Learn how to create and configure menus

## Table of Contents

- [Understanding Extensions and Menus](#understanding-extensions-and-menus)
- [Complete Workflow Example](#complete-workflow-example-from-menu-to-extension-display)
- [Extension Types](#extension-types)
- [Full SDK Access](#full-sdk-access-in-extensions)
- [Complete List of Injected Resources](#complete-list-of-injected-resources)
  - [UI Components](#ui-components-auto-injected)
  - [Enfyra Composables](#enfyra-composables-global-access)
  - [Nuxt Composables](#nuxt-composables-global-access)
  - [Vue 3 Composition API](#vue-3-composition-api-global-access)
  - [Browser APIs](#browser-apis-available)
- [Advanced Extension Features](#advanced-extension-features)
- [Header Actions Integration](#header-actions-integration)
- [Widget System](#widget-system)
- [File Upload Support](#file-upload-support)
- [Extension Management](#extension-management)
- [Best Practices](#best-practices)
- [Common Issues and Solutions](#common-issues-and-solutions)
- [Advanced Patterns](#advanced-patterns)
- [Security Considerations](#security-considerations)
- [Summary](#summary)

## Understanding Extensions and Menus

### The Problem
When you create a menu item with a custom path like `/reports/sales`, clicking it leads to an empty page because there's no code to handle that route.

### The Solution
Extensions provide the Vue.js component that renders when users navigate to your custom menu path. This creates a complete user experience:
1. **Menu** = Navigation entry point
2. **Extension** = The actual page content

## Complete Workflow Example: From Menu to Extension Display

This example shows the complete process from creating a menu to displaying custom content.

### Step 1: Create a Menu Item
1. Navigate to **Settings → Menu**
2. Click **"Create"** to add a new menu
3. Configure your menu:
   - **Type**: Select "Menu" (for regular menu items)
   - **Label**: "Analytics Dashboard"
   - **Path**: `/custom/analytics` (this will be your extension's URL)
   - **Icon**: Choose "lucide:bar-chart-3"
   - **Sidebar**: Select "Dashboard" (so it appears under Dashboard sidebar)
4. Save the menu item

**Result**: You now have a menu entry, but clicking it shows a blank page because there's no extension linked.

### Step 2: Create the Extension
1. Navigate to **Settings → Extensions**
2. Click **"Create Extension"**
3. Fill in the extension details:
   - **Name**: "Analytics Dashboard"
   - **Extension ID**: Auto-generated (e.g., "analytics-dashboard-1234")
   - **Type**: Select "Page" (for menu-linked extensions)
   - **Description**: "Custom analytics dashboard with charts and metrics"
   - **Menu**: Use the relation picker to **select the menu you just created**
   - **Version**: "1.0.0"
   - **Is Enabled**: Check this box

### Step 3: Write Your Extension Code
In the code editor, write your Vue.js Single File Component (SFC):

```vue
<template>
  <div class="p-6 space-y-6">
    <!-- Header -->
    <div class="flex items-center justify-between">
      <div>
        <h1 class="text-3xl font-bold">Analytics Dashboard</h1>
        <p class="text-gray-500">Track your key metrics</p>
      </div>
      <UBadge color="green">Live Data</UBadge>
    </div>
    
    <!-- Stats Cards -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
      <UCard>
        <div class="text-center">
          <div class="text-2xl font-bold text-blue-600">{{ stats.users }}</div>
          <div class="text-sm text-gray-500">Total Users</div>
        </div>
      </UCard>
      
      <UCard>
        <div class="text-center">
          <div class="text-2xl font-bold text-green-600">{{ stats.revenue }}</div>
          <div class="text-sm text-gray-500">Monthly Revenue</div>
        </div>
      </UCard>
      
      <UCard>
        <div class="text-center">
          <div class="text-2xl font-bold text-purple-600">{{ stats.orders }}</div>
          <div class="text-sm text-gray-500">Orders Today</div>
        </div>
      </UCard>
    </div>
    
    <!-- Actions -->
    <div class="flex gap-4">
      <UButton @click="refreshData" :loading="loading" color="primary">
        Refresh Data
      </UButton>
      
      <PermissionGate :condition="{ route: '/reports', actions: ['create'] }">
        <UButton @click="generateReport" variant="outline">
          Generate Report
        </UButton>
      </PermissionGate>
    </div>
    
    <!-- Data Table -->
    <UCard>
      <template #header>
        <h3 class="text-lg font-semibold">Recent Activity</h3>
      </template>
      
      <UTable :rows="recentActivity" :columns="columns" />
    </UCard>
  </div>
</template>

<script setup>
// Vue 3 Composition API - available globally
const { ref, reactive, computed, onMounted } = Vue;

// Enfyra composables - available globally
const toast = useToast();
const { me } = useEnfyraAuth();

// Reactive state
const loading = ref(false);
const stats = reactive({
  users: 1234,
  revenue: '$12,450',
  orders: 89
});

const recentActivity = ref([
  { id: 1, action: 'User login', user: 'john@example.com', time: '2 mins ago' },
  { id: 2, action: 'New order', user: 'jane@example.com', time: '5 mins ago' },
  { id: 3, action: 'Payment received', user: 'bob@example.com', time: '10 mins ago' }
]);

const columns = [
  { key: 'action', label: 'Action' },
  { key: 'user', label: 'User' },
  { key: 'time', label: 'Time' }
];

// Methods
const refreshData = async () => {
  loading.value = true;
  
  try {
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 1000));
    
    // Update stats
    stats.users += Math.floor(Math.random() * 10);
    stats.orders += Math.floor(Math.random() * 5);
    
    toast.add({
      title: 'Data refreshed',
      description: 'Analytics data has been updated',
      color: 'success'
    });
  } catch (error) {
    toast.add({
      title: 'Error',
      description: 'Failed to refresh data',
      color: 'error'
    });
  } finally {
    loading.value = false;
  }
};

const generateReport = () => {
  toast.add({
    title: 'Report generated',
    description: 'Your analytics report is being prepared',
    color: 'info'
  });
};

// Register header actions
onMounted(() => {
  useHeaderActionRegistry([
    {
      id: 'refresh-analytics',
      label: 'Refresh',
      icon: 'lucide:refresh-cw',
      onClick: refreshData,
      permission: {
        route: '/analytics',
        actions: ['read']
      }
    },
    {
      id: 'export-analytics', 
      label: 'Export',
      icon: 'lucide:download',
      variant: 'outline',
      onClick: () => {
        toast.add({
          title: 'Exporting...',
          description: 'Preparing analytics export',
          color: 'info'
        });
      }
    }
  ]);
  
  console.log('Analytics Dashboard loaded for user:', me.value?.email);
});
</script>
```

### Step 4: Save and Test
1. Click **"Create"** to save your extension
2. The system automatically compiles your Vue code
3. Navigate to your custom menu path: `/custom/analytics`
4. **Your extension content now displays!**

### Step 5: The Complete Flow
**What happens when user clicks the menu:**

1. **User clicks** "Analytics Dashboard" in the menu
2. **Browser navigates** to `/custom/analytics`
3. **Enfyra's dynamic router** catches the route
4. **System queries** for menu with path `/custom/analytics`  
5. **Finds linked extension** through the one-to-one relationship
6. **Loads and compiles** the extension Vue SFC code
7. **Renders extension** with full access to components and composables
8. **User sees** the custom analytics dashboard content

**The Magic**: Menu provides navigation → Extension provides content → Complete user experience!

## Extension Types

### Page Extensions
- **Purpose**: Full-page applications linked to menu items
- **Usage**: Selected through menu relation picker
- **Example**: Dashboards, reports, custom forms

### Widget Extensions  
- **Purpose**: Reusable components for embedding anywhere
- **Usage**: Can be embedded using `<Widget :id="extensionId" />`
- **Example**: Charts, status cards, mini-forms

## Full SDK Access in Extensions

**🚀 Complete SDK Integration**: Extensions have full access to all Enfyra SDK features from `@enfyra/sdk-nuxt`. Every composable, utility, and API feature available in the main application is also available in your extensions - no limitations.

## Complete List of Injected Resources

Extensions have access to a comprehensive set of resources that are automatically injected at runtime:

### UI Components (Auto-Injected)
All UI components are automatically injected by the extension system and can be used directly in templates without imports:

**Nuxt UI Components** ([📖 Nuxt UI Documentation](https://ui.nuxt.com/)):
- `UIcon`, `Icon` - Icons and SVG components
- `UButton` - Buttons with variants and states
- `UCard` - Container cards with headers/footers
- `UBadge` - Status badges and labels
- `UInput` - Text input fields
- `UTextarea` - Multi-line text areas
- `USelect` - Dropdown selection
- `UCheckbox` - Checkbox inputs
- `USwitch` - Toggle switches
- `UModal` - Modal dialogs
- `UPopover` - Popover overlays
- `UTooltip` - Hover tooltips
- `UAlert` - Alert messages
- `UAvatar` - User avatars
- `UProgress` - Progress indicators
- `UTable` - Data tables
- `UPagination` - Page navigation
- `UBreadcrumb` - Breadcrumb navigation
- `UTabs` - Tab interfaces
- `UAccordion` - Collapsible content
- `UForm` - Form containers

**Custom Enfyra Components:**
- `DataTable` - Advanced data tables with filtering
- `PermissionGate` - Permission-based content visibility
- `FormEditor` - Dynamic form generation
- `FilterDrawer` - Advanced filtering interface
- `LoadingState` - Loading state indicators
- `EmptyState` - Empty state displays
- `SettingsCard` - Settings interface cards
- `Image` - Enhanced image component
- `UploadModal` - File upload interface
- `Widget` - Dynamic widget embedding

```vue
<template>
  <!-- Components are injected and can be used directly -->
  <UCard class="p-6 space-y-4">
    <!-- Form Elements -->
    <UInput v-model="name" placeholder="Enter name" />
    <UTextarea v-model="description" placeholder="Enter description" />
    <USelect v-model="selectedOption" :options="options" />
    <USwitch v-model="enabled" label="Enable feature" />
    
    <!-- Buttons and Actions -->
    <UButton @click="handleClick" color="primary">
      Click Me
    </UButton>
    
    <!-- Data Display -->
    <UTable :rows="data" :columns="columns" />
    <UBadge color="green">Status: Active</UBadge>
    
    <!-- Advanced Components -->
    <PermissionGate :condition="{ route: '/users', actions: ['read'] }">
      <UButton variant="outline">Admin Only Button</UButton>
    </PermissionGate>
  </UCard>
</template>

<script setup>
// Components are automatically available - no need to import or access through props
// They are injected by the extension system
const { ref, reactive, computed, onMounted } = Vue;

const name = ref('');
const description = ref('');
const enabled = ref(false);
const selectedOption = ref(null);

const options = ['Option 1', 'Option 2', 'Option 3'];
const data = [
  { id: 1, name: 'Item 1', status: 'Active' },
  { id: 2, name: 'Item 2', status: 'Inactive' }
];
const columns = [
  { key: 'id', label: 'ID' },
  { key: 'name', label: 'Name' },
  { key: 'status', label: 'Status' }
];

const handleClick = () => {
  console.log('Button clicked!');
};
</script>
```

### Enfyra Composables (Global Access)
**All Enfyra composables are automatically injected and available globally:**

**API & Data:**
- `useApi()` - Custom API wrapper with error handling (recommended)
- `useEnfyraApi()` - Direct SDK API calls
- `useSchema()` - Schema validation and form generation
- `useFilterQuery()` - Advanced filtering and querying → [Filter System Guide](./filter-system.md)
- `useDataTableColumns()` - Data table column management

**Authentication & Permissions:**
- `useEnfyraAuth()` - Authentication state and methods
- `usePermissions()` - Permission checking and validation

**UI & State Management:**
- `useHeaderActionRegistry()` - Register header actions
- `useSubHeaderActionRegistry()` - Register sub-header actions  
- `useScreen()` - Screen size and responsive utilities
- `useGlobalState()` - Global state management
- `useConfirm()` - Confirmation dialogs

### Nuxt Composables (Global Access)
**All Nuxt composables are available without import:**

**Navigation & Routing:**
- `useRoute()` - Current route information
- `useRouter()` - Router instance for navigation
- `navigateTo()` - Programmatic navigation

**State Management:**
- `useState()` - Nuxt state management
- `useCookie()` - Cookie management

**Data Fetching:**
- `useFetch()` - Server-side data fetching
- `useAsyncData()` - Async data handling
- `useLazyFetch()` - Lazy data loading

**Meta & SEO:**
- `useHead()` - Document head management
- `useSeoMeta()` - SEO metadata

**App Context:**
- `useNuxtApp()` - Nuxt app instance
- `useToast()` - Toast notifications

### Vue 3 Composition API (Global Access)
**Complete Vue 3 Composition API is available globally:**

**Core Reactivity:**
- `ref()` - Create reactive references
- `reactive()` - Create reactive objects
- `computed()` - Computed properties
- `readonly()` - Read-only reactive data
- `shallowRef()` - Shallow reactive references
- `shallowReactive()` - Shallow reactive objects

**Lifecycle Hooks:**
- `onMounted()` - Component mounted
- `onUnmounted()` - Component unmounted
- `onBeforeMount()` - Before component mount
- `onBeforeUnmount()` - Before component unmount
- `onUpdated()` - Component updated
- `onBeforeUpdate()` - Before component update

**Watchers:**
- `watch()` - Watch reactive data
- `watchEffect()` - Effect-based watching

**Component Composition:**
- `defineProps()` - Define component props
- `defineEmits()` - Define component events
- `defineExpose()` - Expose component methods
- `defineComponent()` - Define Vue component
- `h()` - Render function helper
- `resolveComponent()` - Resolve component by name

**Utilities:**
- `nextTick()` - Wait for DOM updates
- `toRef()` - Convert to ref
- `toRefs()` - Convert to refs
- `unref()` - Unwrap ref value
- `isRef()` - Check if value is ref
- `markRaw()` - Mark as non-reactive
- `toRaw()` - Get raw object
- `isProxy()`, `isReactive()`, `isReadonly()` - Type checking
- `effectScope()` - Effect scope management
- `getCurrentScope()` - Get current scope
- `onScopeDispose()` - Scope cleanup

### Browser APIs (Available)
**Standard browser APIs are accessible:**
- `fetch()` - HTTP requests
- `console` - Console logging
- `window` - Window object
- `document` - DOM manipulation

**For complete API usage examples, see [API Integration Guide](./api-integration.md)**

**Basic injected resources usage example:**

```vue
<script setup>
// API Access - Custom wrapper with error handling
const { data } = await useApi('/extension_definition', {
  query: { limit: 10 }
});

// Direct SDK access also available
const { data: directData } = await useEnfyraApi('/extension_definition', {
  query: { limit: 10 }
});

// Authentication - Full SDK access
const { me, isLoggedIn, login, logout } = useEnfyraAuth();

// Permissions - Global composable
const { hasPermission } = usePermissions();
if (hasPermission('/users', 'POST')) {
  // User can create
}

// Notifications - Global composable
const toast = useToast();
toast.add({ 
  title: 'Success!', 
  color: 'success' 
});

// Navigation - Global Nuxt composables
const router = useRouter();
const route = useRoute();

// Schema & Validation - Global composable
const { validate, generateEmptyForm } = useSchema('extension_definition');

// Vue 3 Composition API - Global access
const { ref, reactive, computed, watch, onMounted } = Vue;

// State management - Global Nuxt composables
const globalState = useState('myExtension', () => ({}));
</script>
```

### Permission Gates
Control visibility based on permissions:

```vue
<template>
  <PermissionGate :condition="{ 
    route: '/users', 
    actions: ['create'] 
  }">
    <UButton>Admin Only Button</UButton>
  </PermissionGate>
</template>
```

## Advanced Extension Features

## Header Actions Integration

**🚀 Extensions can inject custom actions directly into the app's header and sub-header areas** - demonstrating the incredible power to intervene in ANY part of the application interface.

### Quick Header Action Example

```vue
<script setup>
onMounted(() => {
  // Register in main header (top-right)
  useHeaderActionRegistry().register({
    id: 'save-report',
    label: 'Save Report',
    icon: 'lucide:save',
    color: 'primary',
    onClick: () => saveReport(),
    permission: {
      route: '/reports',
      actions: ['create']
    }
  });
  
  // Register in sub-header (page level)
  useSubHeaderActionRegistry().register({
    id: 'filter-toggle',
    label: 'Filters',
    icon: 'lucide:filter',
    side: 'left',
    onClick: () => toggleFilters()
  });
});
</script>
```

### Powerful Features
- **Permission Integration**: Every action automatically uses PermissionGate
- **Route Awareness**: Show/hide actions based on current page
- **Custom Components**: Inject complete custom widgets
- **Reactive Properties**: Dynamic labels, loading states, conditional visibility
- **Positioning Control**: Left/right positioning in sub-header

**→ [Complete Header Actions Guide](./header-actions.md)** - Full documentation with advanced examples

### Fetching Data from API
```vue
<script setup>
// Using custom API wrapper (recommended)
const { data: usersData, pending, error, refresh } = await useApi('/user_definition', {
  query: {
    limit: 10,
    fields: 'id,email,name,role.name',
    include: 'role'
  },
  key: 'users-list'
});

// Or direct SDK access for advanced usage
const { data: directData } = await useEnfyraApi('/user_definition', {
  query: { limit: 10 },
  server: true // Server-side rendering option
});

// Computed for easy access
const users = computed(() => usersData.value?.data || []);

// Manual refresh
const loadUsers = () => {
  refresh();
};

// Handle API responses
watch(error, (newError) => {
  if (newError) {
    toast.add({
      title: 'Error loading users',
      description: newError.message,
      color: 'error'
    });
  }
});

onMounted(() => {
  console.log('Users loaded:', users.value.length);
});
</script>

<template>
  <div>
    <!-- Loading state -->
    <div v-if="pending">Loading users...</div>
    
    <!-- Error state -->
    <UAlert v-else-if="error" color="red">
      {{ error.message }}
    </UAlert>
    
    <!-- Data display -->
    <div v-else>
      <div v-for="user in users" :key="user.id">
        {{ user.name }} - {{ user.role?.name }}
      </div>
      
      <UButton @click="loadUsers">Refresh</UButton>
    </div>
  </div>
</template>
```

**See [API Integration](./api-integration.md) for complete API usage guide.**

### Creating Forms

Extensions can use Enfyra's powerful form system to create dynamic, validated forms:

**→ [Complete Form System Guide](./form-system.md)** - Learn about dynamic forms, validation, and field types

```vue
<template>
  <UCard>
    <FormEditor
      :schema="schema"
      v-model="formData"
      @submit="handleSubmit"
    />
  </UCard>
</template>

<script setup>
const schema = await useSchema('products');
const formData = ref(schema.generateEmptyForm());

const handleSubmit = async () => {
  const { isValid, errors } = schema.validate(formData.value);
  
  if (!isValid) {
    toast.add({
      title: 'Validation failed',
      color: 'red'
    });
    return;
  }
  
  await useApi('/products', {
    method: 'POST',
    body: formData.value
  });
};
</script>
```

## Widget System

### Using Widget Extensions
Widgets are reusable components that can be embedded anywhere:

```vue
<template>
  <div class="grid grid-cols-2 gap-4">
    <!-- Embed widget by database ID (not extensionId) -->
    <Widget :id="5" />
    
    <!-- Another widget -->
    <Widget :id="6" />
  </div>
</template>
```

**Important**: Widget `id` is the numeric database ID from the Extensions list, not the `extensionId` string.

### Creating Widget Extensions

1. **Create Extension** with type "Widget":
   - **Name**: Widget display name
   - **Type**: Select "Widget" 
   - **Description**: What this widget does
   - **No menu linking needed** (widgets are embedded, not navigated to)

2. **Write Widget Code**:
```vue
<template>
  <UCard>
    <template #header>
      <h3>Sales Summary</h3>
    </template>
    
    <div class="text-2xl font-bold">
      ${{ totalSales.toLocaleString() }}
    </div>
    <p class="text-gray-500">This month</p>
  </UCard>
</template>

<script setup>
const { ref, computed, onMounted } = Vue;
const totalSales = ref(0);

// Load data using custom wrapper
onMounted(async () => {
  const { data } = await useApi('/sales_summary');
  totalSales.value = data.total;
});
</script>
```

3. **Embed Widget**: Use `<Widget :id="database_id" />` in any extension or page

## File Upload Support
Extensions can handle file uploads:

```vue
<template>
  <input 
    type="file" 
    @change="handleFileUpload"
    accept=".vue"
  />
</template>

<script setup>
const handleFileUpload = async (event) => {
  const file = event.target.files[0];
  if (!file) return;
  
  const content = await file.text();
  // Process the file content
  console.log('File content:', content);
};
</script>
```

## Extension Management

### Enabling/Disabling Extensions
1. Go to **Settings → Extensions**
2. Find your extension in the list
3. Toggle the switch to enable/disable
4. Disabled extensions won't load even if menu is clicked

### Editing Extensions
1. Click on any extension card to open editor
2. Modify the code in the editor
3. Click **"Save"** to recompile
4. Changes take effect immediately

### Version Control
- Update version number when making changes
- System tracks creation and modification timestamps
- User information stored for audit trail

## Best Practices

### Code Organization
```vue
<template>
  <!-- Keep template clean and organized -->
  <div class="extension-container">
    <ExtensionHeader />
    <ExtensionContent />
    <ExtensionFooter />
  </div>
</template>

<script setup>
// 1. Imports and composables
// useApi is available directly - no destructuring needed
const toast = useToast();
const toast = useToast();

// 2. Reactive state
const state = reactive({
  loading: false,
  data: []
});

// 3. Computed properties
const filteredData = computed(() => {
  return state.data.filter(item => item.active);
});

// 4. Methods
const loadData = async () => {
  // Implementation
};

// 5. Lifecycle hooks
onMounted(() => {
  loadData();
});
</script>

<style scoped>
.extension-container {
  @apply p-6;
}
</style>
```

### Error Handling
```vue
<script setup>
const loadData = async () => {
  try {
    const response = await useApi('/endpoint');
    // Handle success
  } catch (error) {
    console.error('Extension error:', error);
    toast.add({
      title: 'Error',
      description: error.message,
      color: 'red'
    });
  }
};
</script>
```

### Performance Tips
- Use `computed` for derived values
- Implement pagination for large datasets
- Lazy load heavy components
- Clean up resources in `onUnmounted`

## Common Issues and Solutions

### Extension Not Loading
**Problem**: Clicking menu shows blank page
**Solution**: 
1. Check extension is enabled
2. Verify extension is linked to correct menu
3. Check browser console for compilation errors
4. Ensure user has permission to access

### Compilation Errors
**Problem**: Extension fails to compile
**Solution**:
1. Check Vue.js syntax is correct
2. Ensure all imported components exist
3. Verify script setup syntax
4. Look for error messages in the form

### Missing Components
**Problem**: Components not recognized
**Solution**:
- Use `props.components.ComponentName` for dynamic access
- Or use global components directly (UButton, UCard, etc.)

### API Calls Failing
**Problem**: Cannot fetch data
**Solution**:
1. Check user has required permissions
2. Verify API endpoint exists
3. Check network tab for errors
4. Ensure proper authentication

## Advanced Patterns

### State Management
```vue
<script setup>
// Use useState for cross-component state
const globalState = useState('myExtension', () => ({
  counter: 0,
  items: []
}));

// Update state
globalState.value.counter++;
</script>
```

### Component Composition
```vue
<script setup>
// Break large extensions into smaller components
const components = {
  Header: {
    template: '<div>Header</div>'
  },
  Footer: {
    template: '<div>Footer</div>'
  }
};
</script>

<template>
  <component :is="components.Header" />
  <component :is="components.Footer" />
</template>
```

### Dynamic Loading
```vue
<script setup>
const dynamicComponent = ref(null);

const loadComponent = async () => {
  // Load component based on conditions
  if (someCondition) {
    dynamicComponent.value = await loadExtension('widget-1');
  }
};
</script>

<template>
  <component :is="dynamicComponent" v-if="dynamicComponent" />
</template>
```

## Security Considerations

### Permission Checks
Always verify permissions in your extensions:

```vue
<script setup>
const { hasPermission } = usePermissions();

// Check before showing sensitive data
const canViewFinancials = computed(() => {
  return hasPermission('/financial_reports', 'GET');
});

// Check before allowing actions
const deleteRecord = async (id) => {
  if (!hasPermission('/records', 'DELETE')) {
    toast.add({
      title: 'Permission denied',
      color: 'red'
    });
    return;
  }
  
  await useApi(`/records/${id}`, { method: 'DELETE' });
};
</script>
```

### Input Validation
Validate user input before sending to API:

```vue
<script setup>
const validateEmail = (email) => {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
};

const submitForm = async () => {
  if (!validateEmail(formData.value.email)) {
    toast.add({
      title: 'Invalid email address',
      color: 'red'
    });
    return;
  }
  
  // Proceed with submission
};
</script>
```

## Summary

The Extension System provides a powerful way to add custom functionality to Enfyra:
1. **Create Menu** → Defines the navigation entry
2. **Create Extension** → Provides the page content  
3. **Link Together** → Menu and extension work as one
4. **Write Vue Code** → Full Vue 3 SFC support with auto-injected components
5. **Full SDK Access** → Complete access to all Enfyra SDK features and composables
6. **Access Resources** → UI components, API, permissions, Vue functions
7. **Deploy Instantly** → No build process required

Extensions give you the flexibility to create any custom functionality while maintaining the security and consistency of the Enfyra platform. With full SDK integration, extensions have the same power as the core application.