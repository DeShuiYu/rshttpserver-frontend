<script setup lang="ts">
import {
  Breadcrumb,
  BreadcrumbEllipsis,
  BreadcrumbItem,
  BreadcrumbLink,
  BreadcrumbList,
  BreadcrumbPage,
  BreadcrumbSeparator,
} from '@/components/ui/breadcrumb'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { House } from 'lucide-vue-next';
import {computed,} from "vue";


const props = defineProps<{
  rootpath: string,
  refreshEntryInfo: (path:string)=>void,
}>()

const emits = defineEmits<{
  'update:rootpath': [value: string]
}>()

const rootPathSplit = computed(() => {
  if (!props.rootpath) return []
  return props.rootpath.split('/').filter(Boolean) // 过滤空字符串
})

function rootPathConcat(start:number,end:number) {
  return rootPathSplit.value.slice(start,end).join("/")
}

function handleChangeRootPath(updateRootPath:string) {
  emits('update:rootpath',updateRootPath)
  props.refreshEntryInfo(updateRootPath)
}

</script>

<template>
  <Breadcrumb>
    <BreadcrumbList>
      <BreadcrumbItem>
        <BreadcrumbLink @click="()=>handleChangeRootPath('')">
          <House />
        </BreadcrumbLink>
      </BreadcrumbItem>
      <BreadcrumbSeparator v-if="rootPathSplit.length > 2"/>
      <BreadcrumbItem v-if="rootPathSplit.length > 2" >
        <DropdownMenu>
          <DropdownMenuTrigger class="flex items-center gap-1">
            <BreadcrumbEllipsis class="h-4 w-4" />
            <span class="sr-only">Toggle menu</span>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="start">
            <DropdownMenuItem
                v-for="(item,key) in rootPathSplit.slice(0,rootPathSplit.length-2)"
                :key="key"
                @click="()=>handleChangeRootPath(rootPathConcat(0,key+1))"
            >
              {{item}}
            </DropdownMenuItem>
          </DropdownMenuContent>
        </DropdownMenu>
      </BreadcrumbItem>
      <BreadcrumbSeparator v-if="rootPathSplit.length > 1"/>
      <BreadcrumbItem v-if="rootPathSplit.length > 1">
        <BreadcrumbLink @click="()=>handleChangeRootPath(rootPathConcat(0,rootPathSplit.length-1))">
          {{rootPathSplit[rootPathSplit.length - 2]}}
        </BreadcrumbLink>
      </BreadcrumbItem>
      <BreadcrumbSeparator v-if="rootPathSplit.length > 0"/>
      <BreadcrumbItem v-if="rootPathSplit.length > 0">
        <BreadcrumbPage >{{rootPathSplit[rootPathSplit.length-1]}}</BreadcrumbPage>
      </BreadcrumbItem>
    </BreadcrumbList>
  </Breadcrumb>
</template>
