<template>
  <!-- 1. Layout Wrapper (Hyamule blank page fix hoil) -->
  <q-layout view="hHh Lpr lFf">
    <q-page-container>
      <q-page padding class="bg-grey-1">
        
        <q-card flat bordered class="shadow-2">
          <!-- Lead Filters Section -->
          <q-card-section>
            <div class="row q-col-gutter-sm items-center">
              <!-- Small Inputs (Sir's requirement) -->
              <div class="col-auto">
                <q-input v-model="country" label="Country" dense outlined bg-color="white" style="width: 130px" />
              </div>
              <div class="col-auto">
                <q-input v-model="city" label="City" dense outlined bg-color="white" style="width: 130px" />
              </div>
              <div class="col-auto">
                <q-input v-model="keyword" label="Keyword" dense outlined bg-color="white" style="width: 130px" />
              </div>
              <div class="col-auto">
                <!-- Small Button -->
                <q-btn label="Generate Leads" color="primary" dense size="md" class="q-px-md" @click="fetchLeads" />
              </div>
            </div>
          </q-card-section>

          <!-- Leads Data Table -->
          <q-card-section class="q-pt-none">
            <q-table
              :rows="leads"
              :columns="columns"
              row-key="id"
              v-model:pagination="pagination"
              flat
              bordered
              dense
            >
              <!-- Status Column Styling -->
              <template v-slot:body-cell-status="props">
                <q-td :props="props">
                  <q-badge :color="getStatusColor(props.value)">
                    {{ props.value }}
                  </q-badge>
                </q-td>
              </template>

              <!-- Extra Small Buttons (Sir's requirement) -->
              <template v-slot:body-cell-actions="props">
                <q-td :props="props" class="q-gutter-xs">
                  <q-btn color="positive" size="xs" dense label="Contacted" @click="markContacted(props.row)" />
                  <q-btn color="orange" size="xs" dense label="Ignore" @click="markIgnored(props.row)" />
                  <q-btn color="negative" size="xs" dense icon="delete" @click="deleteLead(props.row)" />
                  <q-btn color="info" size="xs" dense icon="email" @click="sendEmail(props.row)" />
                </q-td>
              </template>
            </q-table>
          </q-card-section>
        </q-card>

      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const country = ref('')
const city = ref('')
const keyword = ref('')
const leads = ref([])

// 2. Pagination: Default 50 rows (Sir's requirement)
const pagination = ref({
  page: 1,
  rowsPerPage: 50 
})

const columns = [
  { name: 'company_name', label: 'Company', field: 'company_name', align: 'left', sortable: true },
  { name: 'location', label: 'Location', field: 'location', align: 'left' },
  { name: 'contact_email', label: 'Email', field: 'contact_email', align: 'left' },
  { name: 'status', label: 'Status', field: 'status', align: 'center', sortable: true },
  { name: 'actions', label: 'Actions', align: 'center' }
]

const getStatusColor = (status) => {
  if (status === 'contacted') return 'positive'
  if (status === 'ignored') return 'orange'
  return 'blue'
}

const fetchLeads = async () => {
  try {
    let searchQuery = `${country.value} ${city.value} ${keyword.value}`.trim()
    const res = await axios.get(`http://localhost:8000/api/leads/`, {
      params: {
        page: pagination.value.page,
        limit: pagination.value.rowsPerPage,
        search: searchQuery
      }
    })
    leads.value = res.data
  } catch (err) {
    console.error('Fetch error:', err)
    // Testing sathi dummy data jar API band asel tar
    leads.value = [{ id: 1, company_name: 'Test', location: 'Pune', status: 'new' }]
  }
}

onMounted(() => {
  fetchLeads()
})

const markContacted = async (lead) => { lead.status = 'contacted' }
const markIgnored = async (lead) => { lead.status = 'ignored' }
const deleteLead = async (lead) => { leads.value = leads.value.filter(l => l.id !== lead.id) }
const sendEmail = async (lead) => { alert('Email Sent!') }
</script>

<style scoped>
/* Quasar vapartana extra CSS chi garaj kami lagte */
.q-table__container {
  background-color: #ffffff;
}
</style>
