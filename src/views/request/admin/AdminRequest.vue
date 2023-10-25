<template>
  <div class="request-area">
    <header-component :mainTitle="'Tüm Talepler'"></header-component>
    <TContextMenu ref="menu" :model="items" @select="handleMenuItem">
      <template #item="{ props, label }">
        <span style="margin: 8px 10px; cursor:pointer;" v-bind="props.icon" />
        <span style="margin: 8px 0; cursor: pointer;">{{ label }}</span>
      </template>
    </TContextMenu>
    <TDynamicDialog></TDynamicDialog>
    <TDataTable @contextmenu="onCellRightClick" aria-haspopup="true" v-model:selection="selectedRequest"
      :value="requestList" showGridlines paginator :rows="10" tableStyle="width:100%;" filterDisplay="row">
      <TColumn selectionMode="single" headerStyle="width: 3rem"></TColumn>
      <TColumn field="project" header="Proje Adı" style="20%"></TColumn>
      <TColumn field="company" header="Firma" style="20%"></TColumn>
      <TColumn field="title" header="Talep Başlığı" style="20%"></TColumn>
      <TColumn field="desc" header="Talep Açıklama" style="20%"></TColumn>
      <TColumn field="state" header="Talep Durumu" style="20%" :showFilterMenu="false">
        <template #body="slotProps">
          <TTag :value="slotProps.data.state ? 'Talep Alındı' : 'Talep Bekliyor '"
            :severity="getSeverity(slotProps.data)" />
        </template>
        
      </TColumn>
    </TDataTable>
    <TToast></TToast>
  </div>
</template>

<script>
import HeaderComponent from '@/components/HeaderComponent.vue';
import { getFirestore, collection, query, updateDoc, where, getDocs, onSnapshot, orderBy } from 'firebase/firestore';
import { app } from '@/firebase/config';
import { defineAsyncComponent, onMounted, ref, provide } from 'vue';
import { useDialog } from 'primevue/usedialog';
import { toastSuccess } from '@/components/Base/toast';
export default {
  components: { HeaderComponent },
  name: "AdminRequest",
  setup() {
    const menu = ref();
    // hucrede secilen satir
    const selectedRequest = ref();
    const dialog = useDialog();
    const AdminReqPopup = defineAsyncComponent(() => import('@/views/request/admin/AdminRequestPopup.vue'));
    const items = ref([
      { label: "Talebi Yanıtla", icon: 'pi pi-send', command: () => handleMenuItem('send') },
      { separator: true },
      {
        label: "Talebi Al", icon: 'pi pi-check-circle', command: () => handleMenuItem('addRequest')
      }
    ]);
    const firestore = getFirestore(app);
    const requestList = ref([]);
    onMounted(() => {
      getRequestData();
    });

    // Popup'a verimizi gönderdik
    provide("selectedRequest", selectedRequest);

    const getRequestData = async () => {
      const q = query(collection(firestore, "requests"), orderBy("date", 'desc'));
      await getDocs(q).then((snapshot) => {
        snapshot.forEach((item) => {
          requestList.value.push(item.data());
        });
      });
    }

    // Tag stil ayari
    const getSeverity = (request) => {
      switch (request.state) {
        case true:
          return 'success'
        case false:
          return 'danger'
      }
    }

    const selectedUpdateRequest = async () => {
      const q = query(collection(firestore, "requests"), where("id", "==", selectedRequest.value.id));
      onSnapshot(q, (snapshot) => {
        snapshot.forEach((item) => {
          const docDat = item.data();
          const updatedData = { ...docDat, state: true };
          updateDoc(item.ref, updatedData);
        });
      });
    }

    // Context menu elementine tiklandigi durum
    const handleMenuItem = (send) => {

      if (send === 'send') {
        dialog.open(AdminReqPopup, {
          props: {
            header: 'Talebi Yanıtla',
            style: {
              width: '450px'
            },
            modal: true
          }
        });
      } else if (send === 'addRequest') {

        selectedUpdateRequest();
        // Burada veri listesi sifirlandi ve veriyi yeniden çektik
        requestList.value = [];
        getRequestData();
        toastSuccess("Talep durumu alındı");
      }
    }

    const onCellRightClick = (event) => {
      if (selectedRequest.value !== undefined && !selectedRequest.value.state) {
        menu.value.show(event);
      }
    }

    return { requestList, getSeverity, menu, items, selectedRequest, onCellRightClick, handleMenuItem }
  }
}
</script>

<style scoped>
.request-area {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}
</style>