<script setup lang="ts">
interface Details {
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
interface Emit {
  (e: 'submit', value: Details): void
  (e: 'update:isDialogVisible', value: boolean): void
}

interface Props {
  cardDetails?: Details
  isDialogVisible: boolean
}

const props = withDefaults(defineProps<Props>(), {
  cardDetails: () => ({
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
  }),
})

const emit = defineEmits<Emit>()

const cardDetails = ref<Details>(structuredClone(toRaw(props.cardDetails)))

watch(() => props, () => {
  cardDetails.value = structuredClone(toRaw(props.cardDetails))
})

const formSubmit = () => {
  emit('submit', cardDetails.value)
}

const dialogModelValueUpdate = (val: boolean) => {
  emit('update:isDialogVisible', val)
}

console.log('cardDetails')
console.log(cardDetails)
</script>

<template>
  <VDialog
    :width="$vuetify.display.smAndDown ? 'auto' : 600"
    :model-value="props.isDialogVisible"
    @update:model-value="dialogModelValueUpdate"
  >
    <!-- Dialog close btn -->
    <DialogCloseBtn @click="dialogModelValueUpdate(false)" />

    <VCard class="pa-2 pa-sm-10">
      <!-- 👉 Title -->
      <VCardItem class="text-center">
        <VCardTitle>
          <h4 class="text-h4 mb-2">
            {{ props.cardDetails.Status === 'Order Open' ? 'Order Open Cencon' : 'Order Close Cencon' }}
          </h4>
        </VCardTitle>
        <p class="text-body-1 mb-0">
          {{ props.cardDetails.Status === 'Order Open' ? 'Masukan Kode Open Cencon' : 'Masukan Kode Close Cencon' }}
        </p>
      </VCardItem>

      <VCardText class="pt-6">
        <VForm @submit.prevent="() => {}">
          <VRow>
            <!-- 👉 Card Number -->
            <VCol cols="12">
              <AppTextField
                v-model="cardDetails.NOWO"
                label="No Work Order"
                placeholder="1356 3215 6548 7898"
                type="text"
              />
            </VCol>

            <VCol cols="12">
              <AppTextField
                v-model="cardDetails.WSID"
                label="WSID"
                placeholder="1356 3215 6548 7898"
                type="text"
              />
            </VCol>

            <VCol cols="12">
              <AppTextField
                v-model="cardDetails.JReqOpen"
                label="Kode Open Cencon"
                placeholder="1356 3215 6548 7898"
                type="text"
              />
            </VCol>

            <!-- 👉 Card actions -->
            <VCol
              cols="12"
              class="text-center"
            >
              <VBtn
                class="me-4"
                type="submit"
                @click="formSubmit"
              >
                Submit
              </VBtn>
              <VBtn
                color="secondary"
                variant="tonal"
                @click="$emit('update:isDialogVisible', false)"
              >
                Cancel
              </VBtn>
            </VCol>
          </VRow>
        </VForm>
      </VCardText>
    </VCard>
  </VDialog>
</template>
