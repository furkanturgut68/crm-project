<template>
    <PVDialog :header="'Görüşme Notu Oluştur'" :closeDialog="closeDialog" @onSubmit="sendToNotes">
        <template #dialogForm>
            <form @submit.prevent="sendToNotes">
                <div class="row-element">
                    <div class="input-left-element">
                        <label>Firmayı Seçiniz</label>
                        <TDropdown v-model="selectCompany" optionLabel="compName" :options="companyName"></TDropdown>
                    </div>
                    <div class="input-right-element">
                        <label>Görüşme Başlığı</label>
                        <TInputText v-model="interviewTitle"></TInputText>
                    </div>
                </div>

                <div class="row-element">
                    <div class="input-left-element">
                        <label>Görüşme Tarihi</label>
                        <TCalendar v-model="selectDate" dateFormat="dd/mm/yy" :manualInput="false"></TCalendar>
                    </div>
                    <div class="input-right-element">
                        <label>Görüşme Türü</label>
                        <TDropdown v-model="selectInterviewType" :options="interviewType" optionLabel="label"></TDropdown>
                    </div>
                </div>

                <div style="margin: 10px 0;">
                    <label>Notu Ekle</label>
                    <TextArea autoResize rows="3" cols="10" v-model="interviewDetail" style="width: 100%;"></TextArea>
                </div>
                <TToast></TToast>

            </form>
        </template>
    </PVDialog>
</template>

<script>
import PVDialog from '@/components/PVDialog.vue';
import { onMounted, ref } from 'vue';
import customerList from '@/firebase/getCustomer';
import { toastError, toastSuccess } from '@/components/Base/toast';
import addInterviewNotes from '@/firebase/addInterviewNotes';
import { uid } from 'uid';
import { serverTimestamp } from 'firebase/firestore';
import { usePrimeVue } from 'primevue/config';
export default {
    components: { PVDialog },
    name: "InterviewPopup",
    props: {
        closeDialog: {
            type: Function,
            required: true
        }
    },
    setup(props) {
        onMounted(() => {
            const primevue = usePrimeVue();
            primevue.config.locale.dayNamesMin = ["Pz", "Pt", "Sa", "Ça", "Pe", "Cu", "Ct"];
            primevue.config.locale.monthNames = ["Ocak", "Şubat", "Mart", "Nisan", "Mayıs", "Haziran", "Temmuz", "Ağustos", "Eylül", "Ekim", "Kasım", "Aralık"];
        });
        const selectCompany = ref(null);
        const interviewTitle = ref(null);
        const selectDate = ref(null);
        const selectInterviewType = ref(null);
        const interviewDetail = ref(null);
        const companyName = customerList;

        const interviewType = ref([
            { label: "Uzaktan Toplantı" },
            { label: "Telefon Görüşmesi" },
            { label: "Yüzyüze Toplantı" }
        ]);
        const sendToNotes = async () => {
            if (selectCompany.value !== null && interviewTitle.value !== null && selectDate.value !== null && selectInterviewType.value !== null && interviewDetail.value !== null) {
                toastSuccess("Görüşme başarıyla eklendi");
                let data = {
                    id: uid(),
                    company: selectCompany.value.compName,
                    createDate: serverTimestamp(),
                    date: selectDate.value,
                    title: interviewTitle.value,
                    interviewType: selectInterviewType.value.label,
                    detail: interviewDetail.value,
                    cNameTitle:selectCompany.value.compName
                }
                await addInterviewNotes(data);
                setTimeout(() => {
                    props.closeDialog(true);
                }, 1000);
            } else {
                toastError("Lütfen tüm alanları doldurunuz");
            }
        }

        return { sendToNotes, selectCompany, interviewDetail, interviewTitle, selectDate, selectInterviewType, companyName, interviewType }
    }

}
</script>

<style scoped>
label {
    color: black;
    font-weight: bold;
    margin: 10px 0;
}
</style>