<script setup lang="ts">
definePage({
  meta: {
    action: 'read',
    subject: 'Beranda',
    public: false,
  },
})

interface ViewListData {
  Id: number | null
  NOWO: string
  WSID: string
  Tanggal?: string | null // bisa string ISO atau null
  Jam?: string | null // bisa string ISO atau null
  JReqOpen?: string | null
  JReqClose?: string | null
  KDClose?: string | null
  Lokasi?: string | null
  Mesin?: string | null
  NMCABANG?: string | null
}

interface StatusResult {
  text: string
  color: string
}

const widgetData = ref([
  { title: 'In-Store Sales', value: '$5,345', icon: 'tabler-smart-home', desc: '5k orders', change: 5.7 },
  { title: 'Website Sales', value: '$674,347', icon: 'tabler-device-laptop', desc: '21k orders', change: 12.4 },
  { title: 'Discount', value: '$14,235', icon: 'tabler-gift', desc: '6k orders' },
  { title: 'Affiliate', value: '$8,345', icon: 'tabler-wallet', desc: '150 orders', change: -3.5 },
])

const headers = [
  { title: 'No WO', key: 'NOWO' },
  { title: 'WSID', key: 'WSID' },
  { title: 'Tanggal', key: 'Tanggal' },
  { title: 'Jam', key: 'Jam' },
  { title: 'JReq Open', key: 'JReqOpen' },
  { title: 'JReq Close', key: 'JReqClose' },
  { title: 'KD Close', key: 'KDClose' },
  { title: 'Status', key: 'Status' },

  /* { title: 'Lokasi', key: 'Lokasi' }, */
  { title: 'Mesin', key: 'Mesin' },

  /* { title: 'Nama Cabang', key: 'NMCABANG' }, */
  { title: 'Actions', key: 'actions', sortable: false }, // jika masih ada tombol aksi
]

const isNewCardAddDialogVisible = ref(false)

const selectedStatus = ref()
const selectedCategory = ref()
const selectedStock = ref<boolean | undefined>()
const searchQuery = ref('')
const selectedRows = ref([])

const currentOrderDetails = ref<ViewListData>({
  Id: null,
  NOWO: '',
  WSID: '',
  Tanggal: null, // bisa string ISO atau null
  Jam: null, // bisa string ISO atau null
  JReqOpen: null,
  JReqClose: null,
  KDClose: null,
  Lokasi: null,
  Mesin: null,
  NMCABANG: null,
})

const status = ref([
  { title: 'Order Open', value: 'ORDER OPEN' },
  { title: 'Open', value: 'OPEN' },
  { title: 'Order Close', value: 'ORDER CLOSE' },
  { title: 'Close', value: 'CLOSE' },
])

const categories = ref([
  { title: 'Accessories', value: 'Accessories' },
  { title: 'Home Decor', value: 'Home Decor' },
  { title: 'Electronics', value: 'Electronics' },
  { title: 'Shoes', value: 'Shoes' },
  { title: 'Office', value: 'Office' },
  { title: 'Games', value: 'Games' },
])

const stockStatus = ref([
  { title: 'In Stock', value: true },
  { title: 'Out of Stock', value: false },
])

// Data table options
const itemsPerPage = ref(10)
const page = ref(1)
const sortBy = ref()
const orderBy = ref()

// Update data table options
const updateOptions = (options: any) => {
  sortBy.value = options.sortBy[0]?.key
  orderBy.value = options.sortBy[0]?.order
}

const orderStatus = (status: string) => {
  if (status === 'Order Open')
    return { text: 'ORDER OPEN', color: '#FACC15', icon: 'tabler-device-watch' }
  if (status === 'Open')
    return { text: 'OPEN', color: '#10B981', icon: 'tabler-home' }
  if (status === 'Order Close')
    return { text: 'ORDER CLOSE', color: '#3B82F6', icon: 'tabler-device-imac' }
  if (status === 'Close')
    return { text: 'CLOSE', color: '#1D4ED8', icon: 'tabler-shoe' }
}

const { data: orderCenconData, execute: fetchOrders } = await useApi<any>(createUrl('/api/OrderCencon',
  {
    query: {
      filter: searchQuery, // cocok dengan string? filter
      // tglAwal: startDate?.toISOString(), // DateTime → string ISO
      // tglAkhir: endDate?.toISOString(), // DateTime → string ISO
      // cabang: selectedCabang, // string? cabang
      start: page, // halaman
      pageSize: itemsPerPage, // jumlah per halaman
    },
  },
))

// Callback untuk refresh saat ada pesan SignalR
const onMessageReceived = () => {
  fetchOrders()
}

// Setup SignalR
const { messages, newMessageAlert, startConnection } = useSignalR(onMessageReceived)

const orderCencon = computed((): ViewListData[] => orderCenconData.value.Data)
const totalDataOrder = computed(() => orderCenconData.value.TotalCount)

const prosesOrder = async (data: ViewListData) => {
  console.log(data)
  currentOrderDetails.value = data
  isNewCardAddDialogVisible.value = !isNewCardAddDialogVisible.value

  // await $api(`apps/ecommerce/listOrderCencon/${id}`, {
  //   method: 'DELETE',
  // })

  // // Delete from selectedRows
  // const index = selectedRows.value.findIndex(row => row === id)
  // if (index !== -1)
  //   selectedRows.value.splice(index, 1)

  // // Refetch listOrderCencon
  console.log(currentOrderDetails)
  fetchOrders()
}

const formatTanggal = (tgl: string | null | undefined) => {
  if (!tgl)
    return '-'
  const date = new Date(tgl)
  const day = String(date.getDate()).padStart(2, '0')
  const month = String(date.getMonth() + 1).padStart(2, '0') // 0 = Jan
  const year = date.getFullYear()

  return `${day}-${month}-${year}`
}

const formatJam = (jam: string | null | undefined) => {
  if (!jam)
    return '-'
  const date = new Date(jam)
  const hours = String(date.getHours()).padStart(2, '0')
  const minutes = String(date.getMinutes()).padStart(2, '0')
  const seconds = String(date.getSeconds()).padStart(2, '0')

  return `${hours}:${minutes}:${seconds}`
}

startConnection()

const notificationSound = new Audio('sound/marimba.mp3')

const enableAudio = () => {
  notificationSound.play().catch(() => {}) // unlock audio
}
</script>

<template>
  <div>
    <!-- 👉 listOrderCencon -->
    <VCard
      title="Filters"
      class="mb-6"
    >
      <VCardText>
        <VRow>
          <VCol
            cols="12"
            sm="4"
          >
            <AppSelect
              v-model="selectedStatus"
              placeholder="Status"
              :items="status"
              clearable
              clear-icon="tabler-x"
            />
          </VCol>

          <!--
            <VCol
            cols="12"
            sm="4"
            >
            <AppSelect
            v-model="selectedCategory"
            placeholder="Category"
            :items="categories"
            clearable
            clear-icon="tabler-x"
            />
            </VCol>

            <VCol
            cols="12"
            sm="4"
            >
            <AppSelect
            v-model="selectedStock"
            placeholder="Stock"
            :items="stockStatus"
            clearable
            clear-icon="tabler-x"
            />
            </VCol>
          -->
        </VRow>
      </VCardText>

      <VDivider />

      <div class="d-flex flex-wrap gap-4 ma-6">
        <div class="d-flex align-center">
          <!-- 👉 Search  -->
          <AppTextField
            v-model="searchQuery"
            placeholder="Cari Order Cencon"
            style="inline-size: 200px;"
            class="me-3"
          />
        </div>

        <VSpacer />
        <div class="d-flex gap-4 flex-wrap align-center">
          <AppSelect
            v-model="itemsPerPage"
            :items="[5, 10, 20, 25, 50]"
          />
          <VBtn
            color="primary"
            prepend-icon="tabler-volume"
            @click="enableAudio"
          >
            Check Sound
          </VBtn>
        </div>
      </div>

      <VDivider class="mt-4" />

      <!-- 👉 Datatable  -->
      <VDataTableServer
        v-model:items-per-page="itemsPerPage"
        v-model:model-value="selectedRows"
        v-model:page="page"
        :headers="headers"
        show-select
        :items="orderCencon"
        :items-length="totalDataOrder"
        class="text-no-wrap"
        @update:options="updateOptions"
      >
        <!-- product  -->
        <template #item.NOWO="{ item }">
          <div class="d-flex align-center gap-x-4">
            <div class="d-flex flex-column">
              <span class="text-body-1 font-weight-medium text-high-emphasis">{{ item.NOWO }}</span>
              <span class="text-body-2">{{ item.NMCABANG }}</span>
            </div>
          </div>
        </template>

        <!-- stock -->
        <template #item.WSID="{ item }">
          <span class="text-body-2 text-high-emphasis">{{ item.WSID }}</span>
        </template>

        <!-- stock  -->
        <template #item.Tanggal="{ item }">
          <span class="text-body-2">{{ formatTanggal(item.Tanggal) }}</span>
        </template>

        <template #item.Jam="{ item }">
          <span class="text-body-2">{{ formatJam(item.Jam) }}</span>
        </template>

        <template #item.Status="{ item }">
          <VChip
            v-bind="orderStatus(item.Status)"
            label
            size="small"
          />
        </template>

        <!--
          <template #item.Lokasi="{ item }">
          <span class="text-body-2">{{ item.Lokasi }}</span>
          </template>
        -->

        <template #item.Mesin="{ item }">
          <div class="d-flex align-center gap-x-4">
            <div class="d-flex flex-column">
              <span class="text-body-1 font-weight-medium text-high-emphasis">{{ item.Mesin }}</span>
              <span class="text-body-2">{{ item.Lokasi }}</span>
            </div>
          </div>
          <!-- <span class="text-body-2">{{ item.Mesin }}</span> -->
        </template>

        <!--
          <template #item.NMCABANG="{ item }">
          <span class="text-body-2">{{ item.NMCABANG }}</span>
          </template>
        -->

        <!-- Actions -->
        <template #item.actions="{ item }">
          <IconBtn>
            <VIcon
              icon="tabler-edit"
              @click="prosesOrder(item)"
            />
          </IconBtn>
        </template>

        <!-- pagination -->
        <template #bottom>
          <TablePagination
            v-model:page="page"
            :items-per-page="itemsPerPage"
            :total-items="totalDataOrder"
          />
        </template>
      </VDataTableServer>
    </VCard>
  </div>

  <CardUpdateOrderCencon
    v-model:is-dialog-visible="isNewCardAddDialogVisible"
    :card-details="currentOrderDetails"
  />
</template>
