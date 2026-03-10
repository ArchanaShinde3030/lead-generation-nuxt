<template>
  <div class="lead-page">
    <div class="lead-box">
      <h1 class="title">Lead Generator</h1>

      <!-- Inputs stacked vertically
      <div class="inputs">
        <input v-model="country" placeholder="Country" class="input-field"/>
        <input v-model="city" placeholder="City" class="input-field"/>
        <input v-model="keyword" placeholder="Keyword" class="input-field"/>
        <button @click="fetchLeads" class="btn-generate">Generate Leads</button>
      </div> -->

      <!-- Filters -->
      <div class="filters">
        <select v-model="filterStatus" class="filter-field">
          <option value="">All Status</option>
          <option value="new">New</option>
          <option value="contacted">Contacted</option>
          <option value="ignored">Ignored</option>
        </select>

        <input v-model="filterTech" placeholder="Tech Stack" class="filter-field"/>
        <input v-model="filterLocation" placeholder="Location" class="filter-field"/>
      </div>

      <!-- Loading/Error -->
      <div v-if="loading" class="loading">Loading...</div>
      <div v-if="error" class="error">{{ error }}</div>

      <!-- Leads Table -->
      <div v-if="filteredLeads.length" class="table-wrapper">
        <table class="leads-table">
          <thead>
            <tr>
              <th>Company</th>
              <th>Job Title</th>
              <th>Location</th>
              <th>Tech</th>
              <th>Email</th>
              <th>Status</th>
              <th>Source</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="lead in filteredLeads" :key="lead.id">
              <td>{{ lead.company_name }}</td>
              <td>{{ lead.job_title }}</td>
              <td>{{ lead.location }}</td>
              <td>{{ lead.tech_stack }}</td>
              <td>{{ lead.contact_email || '-' }}</td>
              <td><span :class="statusClass(lead.status)">{{ lead.status }}</span></td>
              <td><a :href="lead.source_url" target="_blank" class="source-link">Link</a></td>
              <td class="actions">
                <button @click="markContacted(lead)" class="btn-contacted">Contacted</button>
                <button @click="markIgnored(lead)" class="btn-ignored">Ignore</button>
                <button @click="deleteLead(lead)" class="btn-delete">Delete</button>
                <button @click="sendEmail(lead)" class="btn-email">Email</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div v-else class="no-leads">No leads found.</div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import axios from 'axios'

const country = ref('')
const city = ref('')
const keyword = ref('')
const leads = ref([])
const loading = ref(false)
const error = ref('')

// Filters
const filterStatus = ref('')
const filterTech = ref('')
const filterLocation = ref('')

// Fetch Leads
const fetchLeads = async () => {
  loading.value = true
  error.value = ''
  try {
    let searchQuery = `${country.value} ${city.value} ${keyword.value}`.trim()
    const res = await axios.get(`http://localhost:8000/api/leads/?search=${encodeURIComponent(searchQuery)}`)
    leads.value = res.data
  } catch (err) {
    error.value = 'Failed to fetch leads'
  } finally {
    loading.value = false
  }
}

onMounted(fetchLeads)

const filteredLeads = computed(() => {
  return leads.value.filter(lead => {
    return (
      (!filterStatus.value || lead.status === filterStatus.value) &&
      (!filterTech.value || (lead.tech_stack || '').toLowerCase().includes(filterTech.value.toLowerCase())) &&
      (!filterLocation.value || (lead.location || '').toLowerCase().includes(filterLocation.value.toLowerCase()))
    )
  })
})

// Actions
const markContacted = async (lead) => {
  await axios.patch(`http://localhost:8000/api/leads/${lead.id}/`, { status: 'contacted' })
  lead.status = 'contacted'
}

const markIgnored = async (lead) => {
  await axios.patch(`http://localhost:8000/api/leads/${lead.id}/`, { status: 'ignored' })
  lead.status = 'ignored'
}

const deleteLead = async (lead) => {
  await axios.delete(`http://localhost:8000/api/leads/${lead.id}/`)
  leads.value = leads.value.filter(l => l.id !== lead.id)
}

const sendEmail = async (lead) => {
  try {
    await axios.post(`http://localhost:8000/api/send-email/${lead.id}/`)
    alert("Email sent")
  } catch (err) {
    alert("Email failed")
  }
}

// Status badge classes
const statusClass = (status) => {
  if(status === 'new') return 'status-new'
  if(status === 'contacted') return 'status-contacted'
  if(status === 'ignored') return 'status-ignored'
  return ''
}
</script>

<style scoped>
/* Full-screen professional container */
.lead-page {
  min-height: 100vh;
  background-color: #f9fafb;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 40px 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.lead-box {
  width: 100%;
  max-width: 1200px;
  background-color: #fff;
  padding: 30px 40px;
  border-radius: 12px;
  box-shadow: 0 15px 35px rgba(0,0,0,0.1);
}

.title {
  font-size: 32px;
  font-weight: bold;
  text-align: center;
  margin-bottom: 30px;
}

/* Input fields stacked vertically */
.inputs {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 25px;
}

.input-field {
  padding: 10px 15px;
  border-radius: 8px;
  border: 1px solid #d1d5db;
  font-size: 15px;
  width: 100%;
}

.input-field:focus {
  outline: none;
  border-color: #f97316;
  box-shadow: 0 0 6px rgba(249,115,22,0.2);
}

/* Generate Leads button */
.btn-generate {
  padding: 12px 0;
  background-color: #f97316;
  color: #fff;
  font-size: 16px;
  font-weight: 600;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: transform 0.2s ease, background 0.2s ease;
}

.btn-generate:hover {
  background-color: #ea580c;
  transform: translateY(-2px);
}

/* Filters */
.filters {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
  margin-bottom: 20px;
}

.filter-field {
  flex: 1;
  padding: 8px 12px;
  border-radius: 8px;
  border: 1px solid #d1d5db;
  font-size: 14px;
}

.filter-field:focus {
  border-color: #f97316;
  box-shadow: 0 0 6px rgba(249,115,22,0.2);
  outline: none;
}

/* Table */
.table-wrapper {
  overflow-x: auto;
}

.leads-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
}

.leads-table th, .leads-table td {
  padding: 10px 12px;
  border: 1px solid #e5e7eb;
  font-size: 14px;
  text-align: left;
}

.leads-table th {
  background-color: #f3f4f6;
  font-weight: 600;
}

/* Status badges */
.status-new { color: #2563eb; font-weight: bold; }
.status-contacted { color: #16a34a; font-weight: bold; }
.status-ignored { color: #dc2626; font-weight: bold; }

/* Source link */
.source-link {
  color: #2563eb;
  text-decoration: underline;
  font-size: 13px;
}

/* Action buttons */
.actions {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
}

.actions button {
  padding: 6px 10px;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  border: none;
  cursor: pointer;
  transition: transform 0.2s ease;
}

.btn-contacted { background-color: #16a34a; color: white; }
.btn-ignored { background-color: #facc15; color: white; }
.btn-delete { background-color: #dc2626; color: white; }
.btn-email { background-color: #f97316; color: white; }

.actions button:hover { transform: translateY(-2px); }

/* Loading / Error / No leads */
.loading, .error, .no-leads {
  text-align: center;
  font-weight: 500;
  margin: 15px 0;
}

.error { color: #dc2626; }
.no-leads { color: #6b7280; }
</style>