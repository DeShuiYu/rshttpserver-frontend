<script setup lang="ts">
import {Button} from '@/components/ui/button'
import {
  Dialog,
  DialogClose,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog'
import {Input} from '@/components/ui/input'
import {Label} from '@/components/ui/label'
import {FileUp, Upload, X, FileText} from 'lucide-vue-next';
import {ref} from "vue";
import {toast} from "vue-sonner";

const props = defineProps<{
  rootpath: string,
  refreshEntryInfo: (path:string)=>void,
}>()


// 1. 定义状态
const isOpen = ref(false)
const isLoading = ref(false)
const selectedFiles = ref<File[]>([])
// 处理文件选择
const handleFileChange = (event: Event) => {
  const target = event.target as HTMLInputElement
  if (target.files && target.files.length > 0) {
    // 将 FileList 转换为数组并追加到当前列表中 (如果想要覆盖，直接赋值即可)
    selectedFiles.value = [...selectedFiles.value, ...Array.from(target.files)]

    // 清空 input value，以允许重复选择同一个文件
    target.value = ''
  }
}
// 移除单个文件
const removeFile = (index: number) => {
  selectedFiles.value.splice(index, 1)
}

// 格式化文件大小 (辅助函数)
const formatSize = (bytes: number) => {
  if (bytes === 0) return '0B'
  const k = 1024
  const sizes = ['B', 'K', 'M', 'G']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return (bytes / Math.pow(k, i)).toFixed(2) + sizes[i]
}
// 2. 定义点击 Upload 时的处理函数
const handleSave = async () => {
  if (selectedFiles.value.length === 0) return

  try {
    isLoading.value = true
    console.log('Start uploading files...')

    // --- 模拟 FormData 构建 ---
    const formData = new FormData()
    selectedFiles.value.forEach((file) => {
      formData.append('files', file)
    })

    // 打印看看 (实际开发中这里直接发送 axios/fetch 请求)
    console.log('Uploading:', selectedFiles.value.map(f => f.name))


    const url = `/api/upload/${props.rootpath?.trim() ? props.rootpath.trim() + '/' : ''}`
    console.log("AppUploadFiles url:", url)
    // 模拟异步 API 请求 (等待 1.5 秒)
    // await new Promise((resolve) => setTimeout(resolve, 1500))
    await fetch(url, {
      method: 'POST',
      body: formData,
      // 注意：不要手动设置 Content-Type，浏览器会自动设置 multipart/form-data 和 boundary
    }).then(res => res.json())
        .then(data => {
          toast.success(data.message)
        })
        .catch(error => toast.error(error))

    // 上传成功后关闭弹窗并重置
    isOpen.value = false
    selectedFiles.value = []
  } catch (error) {
    console.error('Upload failed:', error)
  } finally {
    isLoading.value = false
  }
  props.refreshEntryInfo(props.rootpath)
}
</script>

<template>
  <Dialog v-model:open="isOpen">
    <form>
      <DialogTrigger as-child>
        <Button variant="outline">
          <FileUp/>
          Upload Files
        </Button>
      </DialogTrigger>
      <DialogContent class="sm:max-w-[425px]">
        <DialogHeader>
          <DialogTitle>Upload Files</DialogTitle>
          <DialogDescription>
            upload files in the current location.
          </DialogDescription>
        </DialogHeader>
        <div class="grid gap-4">
          <div class="grid gap-3">
            <Label for="file-upload">select files</Label>
            <Input id="file-upload" type="file" multiple @change="handleFileChange"/>
          </div>

          <div v-if="selectedFiles.length > 0" class="max-h-[200px] overflow-y-auto space-y-2">
            <div
                v-for="(file, index) in selectedFiles"
                :key="index + file.name"
                class="flex items-center justify-between p-2 border rounded-md text-sm"
            >
              <div class="flex items-center gap-2 overflow-hidden">
                <FileText class="h-4 w-4 text-blue-500 flex-shrink-0"/>
                <div class="flex flex-col truncate">
                  <span class="truncate font-medium">{{ file.name }}</span>
                  <span class="text-xs text-gray-500">{{ formatSize(file.size) }}</span>
                </div>
              </div>

              <Button
                  variant="ghost"
                  size="icon"
                  class="h-6 w-6 text-gray-500 hover:text-red-500"
                  @click="removeFile(index)"
              >
                <X class="h-4 w-4"/>
              </Button>
            </div>
          </div>

          <div v-else class="text-center py-4 text-sm text-gray-500 border-dashed border-2 rounded-md">
            No files selected
          </div>
        </div>
        <DialogFooter>
          <DialogClose as-child>
            <Button variant="outline">
              Cancel
            </Button>
          </DialogClose>
          <Button
              type="submit"
              @click="handleSave"
              :disabled="isLoading || selectedFiles.length === 0"
          >
            <Upload v-if="!isLoading" class="mr-2 h-4 w-4"/>
            {{ isLoading ? 'Uploading...' : 'Upload' }}
          </Button>
        </DialogFooter>
      </DialogContent>
    </form>
  </Dialog>
</template>
