<template>
  <q-layout view="hHh Lpr lFf">
    <q-page-container>
      <q-page padding class="bg-grey-1">
        
        <q-card flat bordered class="shadow-2">
          <!-- Filters Section -->
          <q-card-section>
            <div class="row q-col-gutter-sm items-center">
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
                <q-btn label="Generate Leads" color="primary" dense size="md" class="q-px-md" @click="fetchLeads" />
              </div>
            </div>
          </q-card-section>

          <!-- Leads Data Table -->
          <q-card-section class="q-pt-none">
            <q-table :rows="leads" :columns="columns" row-key="id" v-model:pagination="pagination" flat bordered dense>
              
              <!-- 1. Source URL Column (Click to Open) -->
              <template v-slot:body-cell-source="props">
                <q-td :props="props">
                  <q-btn 
                    v-if="props.value && props.value !== 'N/A'" 
                    flat 
                    dense 
                    color="blue-8" 
                    icon="launch" 
                    label="Visit" 
                    size="sm"
                    @click="openSource(props.value)"
                  />
                  <span v-else class="text-grey-5">N/A</span>
                </q-td>
              </template>

              <!-- 2. Status Badge -->
              <template v-slot:body-cell-status="props">
                <q-td :props="props">
                  <q-badge :color="getStatusColor(props.value)">{{ props.value }}</q-badge>
                </q-td>
              </template>

              <!-- 3. Actions (With Email Button) -->
              <template v-slot:body-cell-actions="props">
                <q-td :props="props" class="q-gutter-xs text-center">
                  <q-btn color="positive" size="xs" dense label="Contacted" @click="markContacted(props.row)" />
                  <q-btn color="orange" size="xs" dense label="Ignore" @click="markIgnored(props.row)" />
                  <q-btn color="info" size="xs" dense icon="email" @click="sendEmail(props.row)">
                    <q-tooltip>Send Email</q-tooltip>
                  </q-btn>
                  <q-btn color="negative" size="xs" dense icon="delete" @click="deleteLead(props.row)" />
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

const pagination = ref({ page: 1, rowsPerPage: 50 })

const columns = [
  { name: 'company_name', label: 'Company', field: 'company_name', align: 'left', sortable: true },
  { name: 'location', label: 'Location', field: 'location', align: 'left' },
  { name: 'source', label: 'Source', field: 'source_url', align: 'left' }, 
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
    const res = await axios.get(`https://archanashinde-leadgen-backend.hf.space/api/leads/`, {
      params: { search: searchQuery }
    })
    leads.value = res.data
  } catch (err) {
    console.error('Fetch error:', err)
    // Testing dummy data
    leads.value = [{ id: 1, company_name: 'Test Co', location: 'USA', source_url: 'https://example.com', contact_email: 'test@mail.com', status: 'new' }]
  }
}

const openSource = (url) => {
  if (url) window.open(url, '_blank')
}

const sendEmail = async (lead) => {
  try {
    const res = await axios.post(`https://archanashinde-leadgen-backend.hf.space/api/send-email/${lead.id}/`)
    alert(res.data.message)
  } catch (err) {
    console.error("Error:", err.response.data)
    alert(err.response.data.error)
  }
}

const markContacted = (lead) => { lead.status = 'contacted' }
const markIgnored = (lead) => { lead.status = 'ignored' }
const deleteLead = (lead) => { leads.value = leads.value.filter(l => l.id !== lead.id) }

onMounted(() => { fetchLeads() })
</script>



<style scoped>
.q-table__container {
  background-color: #ffffff;
}
</style>
