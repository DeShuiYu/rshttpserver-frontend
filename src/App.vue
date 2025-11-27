<script setup lang="ts">
import AppNavBar from "@/components/AppNavBar.vue";
import AppBreadCrumb from "@/components/AppBreadCrumb.vue";
import { ref, watch} from "vue";
import AppCreateFolder from "@/components/AppCreateFolder.vue";
import AppUploadFiles from "@/components/AppUploadFiles.vue";
import 'vue-sonner/style.css'
import {toast, Toaster} from 'vue-sonner'
import AppFileDataTable from "@/components/AppFileDataTable.vue";
import {formatBytes} from "@/lib/utils.ts";


const rootPath = ref("");
const entriesData = ref([])


function refreshEntryInfo(path: string) {
  console.log(path);
  const url = `/api/info/${rootPath.value?.trim() ? rootPath.value.trim() + '/' : ''}`
  fetch(url, {
    method: "GET",
  })
      .then(res => res.json())
      .then(data => {
        entriesData.value = data.data.map((item:any ,index:number) => ({
          id: index + 1,                    // 通常从 1 开始，或者用 item.id 更好
          ename: item.ename,
          eaccessed: item.eaccessed,
          ecreated:item.ecreated,
          emodified:item.emodified,
          epath:item.epath,
          esize:formatBytes(item.esize),
          eppath:item.eppath,
          etype:item.etype,
        }))
      })
      .catch(error => {
        toast.error(error.message)
      })

  console.log("success refresh entries info")
}

watch(() => rootPath, (newValue) => {
  refreshEntryInfo(newValue.value);
}, {immediate: true});

</script>

<template>
  <div class="main">
    <Toaster position="top-right" richColors/>
    <div class="maincontainer">
      <div class="w-full h-20 flex items-center shadow-md rounded-md px-1">
        <AppNavBar/>
      </div>
      <div class="w-full h-12  flex flex-row items-center shadow-md rounded-md px-1">
        <div class="flex-2">
          <AppBreadCrumb v-model:rootpath="rootPath" :refreshEntryInfo="refreshEntryInfo"/>
        </div>
        <div class="flex-1 flex flex-row justify-end gap-3 ">
          <AppCreateFolder v-model:rootpath="rootPath" :refreshEntryInfo="refreshEntryInfo"/>
          <AppUploadFiles v-model:rootpath="rootPath" :refreshEntryInfo="refreshEntryInfo"/>
        </div>
      </div>
      <div class="w-full h-[calc(100vh-5rem-3rem)] ">
        <AppFileDataTable v-model:data="entriesData" v-model:rootpath="rootPath" :refreshEntryInfo="refreshEntryInfo"/>
      </div>
    </div>

  </div>

</template>

<style scoped lang="css">
@reference "./style.css";
.main {
  @apply flex flex-col w-screen h-screen items-center justify-center ;
}

.maincontainer {
  @apply flex flex-col w-7/10 h-screen items-center justify-start gap-3;
}
</style>
