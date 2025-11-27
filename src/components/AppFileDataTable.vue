<script setup lang="ts">
import type {ColumnDef} from '@tanstack/vue-table'
import {
  FlexRender,
  getCoreRowModel,
  useVueTable,
} from '@tanstack/vue-table'

import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table'
import {h} from "vue";
import {formatTimestamp} from "@/lib/utils.ts";
import {Button} from "@/components/ui/button";
import {Folder, File, FolderSymlink} from 'lucide-vue-next';
import {toast} from "vue-sonner";

const props = defineProps<{
  data: any[],
  rootpath: string,
  refreshEntryInfo: (path:string)=>void,
}>()

const emits = defineEmits<{
  'update:data': [value: any[]]
  'update:rootpath': [value: string]
}>()


function handleDelete(index: number, path: string) {
  if (path === '' || path === undefined || path === '/') {
    console.warn("handleDelete ", path, " is missing")
    return
  }

  const url = `/api/delete/${path.trim() ? path.trim() : ''}`
  fetch(url, {
    method: "DELETE",
  })
      .then(res => res.json())
      .then(data => {
        toast.success(data.message)
      })
      .catch(e => toast.error(e))
  emits('update:data',( props.data)?.toSpliced(index, 1))
}

function handleFolderName(path: string) {
  // emits("update:rootpath",cepath)
  // const url = `/api/info/${path.trim() ? path.trim() : ''}`
  emits("update:rootpath",path)
  props.refreshEntryInfo(path)
}
// function handleDownloadFile(path: string) {
//   const url = `/api/download/${path.trim() ? path.trim() : ''}`
//   console.log("handleDownloadFile:",url)
// }

const columns: ColumnDef<any>[] = [
  {
    id: 'row-index',
    header: "",
    cell: ({ row }) => h('div', {
      class: 'text-center font-medium text-muted-foreground tabular-nums',
    }, row.index),
  },
  {
    accessorKey: 'ename',
    header: "Name",
    meta: {fixed: true},
    cell: ({row}) => {
      const cename = row.getValue('ename')
      const cetype = row.getValue('etype');
      const cepath = row.getValue('epath');

      if (cetype === 'd') {
        return h('div', {
          class: 'text-left font-medium flex flex-row gap-1 hover:underline',
          onClick: () => {
            handleFolderName(cepath as string)
          },
        }, [h(Folder, {class: "size-6"}), cename as string])
      } else if (cetype === 'f') {
        return h('a', {
          class: 'text-left font-medium flex flex-row gap-1 hover:underline',
          href: `/api/download/${cepath}`,
          download:cename,
          // 可选：防止跳转到新页面（推荐加上）
          onClick: (e: Event) => {
            e.stopPropagation()  // 防止触发行点击事件
          },
        }, [h(File, {class: "size-6"}), cename as string])
      } else if (cetype === 'l') {
        return h('div', {
          class: 'text-left font-medium flex flex-row gap-1 hover:underline',
          onClick: () => {
            handleFolderName(cepath as string)
          },
        }, [h(FolderSymlink, {class: "size-6"}), cename as string])
      }else{
        return h('div', {
          class: 'text-left font-medium flex flex-row gap-1 ',
        }, cename as string)
      }
    },
  },
  {
    accessorKey: 'etype',
    header: ()=>h('div', {class: 'text-center font-medium'}, "Type"),
    cell: ({row}) => {
      if (row.getValue('etype') === 'f') {
        return h('div', {class: 'text-center font-medium'}, "F")
      } else if (row.getValue('etype') === 'd') {
        return h('div', {class: 'text-center font-medium'}, "D")
      } else if (row.getValue('etype') === 'l') {
        return h('div', {class: 'text-center font-medium'}, "L")
      }
    },
  },

  {
    accessorKey: 'esize',
    header: ()=>h('div', {class: 'text-center font-medium'}, "Size"),
    cell: ({row}) => {
      const cesize = row.getValue('esize')
      return h('div', {class: 'text-center font-medium'}, cesize as string)
    },
  },
  {
    accessorKey: 'emodified',
    header: "Modified Time",
    cell: ({row}) => {
      const formatTimestampEmodified = formatTimestamp(row.getValue('emodified'))
      return h('div', {class: 'text-left font-medium'}, formatTimestampEmodified)
    },
  },
  {
    accessorKey: 'ecreated',
    header: "Created Time",
    cell: ({row}) => {
      const formatTimestampEcreated = formatTimestamp(row.getValue('ecreated'))
      return h('div', {class: 'text-left font-medium'}, formatTimestampEcreated)
    },
  },
  {
    accessorKey: 'eaccessed',
    header: "Accessed Time",
    cell: ({row}) => {
      const formatTimestampEaccessed = formatTimestamp(row.getValue('eaccessed'))
      return h('div', {class: 'text-left font-medium'}, formatTimestampEaccessed)
    },
  },
  {
    accessorKey: 'epath',
    header: "Path",
    cell: ({row}) => {
      const epath = row.getValue('epath')
      return h('div', {class: 'text-left font-medium'}, epath as string)
    },
  },
  {
    accessorKey: '',
    header: "Actions",
    cell: ({row}) => {
      const cepath = row.getValue('epath')
      const cindex = row.index;
      return h(Button, {
        variant: "destructive",
        onClick: () => handleDelete(cindex, cepath as string),
      }, "Del")
    }
  }

]


const table = useVueTable({
  get data() {
    return props.data
  },
  get columns() {
    return columns
  },
  getCoreRowModel: getCoreRowModel(),
})
</script>

<template>
  <div class="border rounded-md">
    <Table>
      <TableHeader>
        <TableRow v-for="headerGroup in table.getHeaderGroups()" :key="headerGroup.id">
          <TableHead v-for="header in headerGroup.headers"
                     :key="header.id"
                     :class="[
              // 动态应用固定样式
              (header.column.columnDef.meta as any)?.fixed
                ? 'sticky left-0 z-20 bg-background hover:bg-gray-50'
                : ''
            ]"
          >
            <FlexRender
                v-if="!header.isPlaceholder" :render="header.column.columnDef.header"
                :props="header.getContext()"
            />
          </TableHead>
        </TableRow>
      </TableHeader>
      <TableBody>
        <template v-if="table.getRowModel().rows?.length">
          <TableRow
              v-for="row in table.getRowModel().rows" :key="row.id"
              :data-state="row.getIsSelected() ? 'selected' : undefined"
          >
            <TableCell v-for="cell in row.getVisibleCells()"
                       :key="cell.id"
                       :class="[
              // 动态应用固定样式（与 header 一致）
              (cell.column.columnDef.meta as any)?.fixed
                ? 'sticky left-0 z-10 bg-background hover:bg-gray-50'
                : ''
            ]"
            >
              <FlexRender :render="cell.column.columnDef.cell" :props="cell.getContext()"/>
            </TableCell>
          </TableRow>
        </template>
        <template v-else>
          <TableRow>
            <TableCell :colspan="columns.length" class="h-24 text-center">
              No results.
            </TableCell>
          </TableRow>
        </template>
      </TableBody>
    </Table>
  </div>
</template>
