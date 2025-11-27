<script setup lang="ts">
import { Button } from '@/components/ui/button'
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
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { FolderPlus } from 'lucide-vue-next';
import {ref} from "vue";
import { toast } from 'vue-sonner'
const props = defineProps<{
  rootpath: string,
  refreshEntryInfo: (path:string)=>void,
}>()

const isOpen = ref(false);
const foldername = ref<string>('');

function handleSubmitButton() {
  if (isOpen.value) {
      isOpen.value = false;
  }

  const url = `/api/create/${props.rootpath?.trim() ? props.rootpath.trim() + '/' : ''}${foldername.value.trim()}`
  fetch(url)
      .then(res => res.json())
      .then(data => {
        if (data.code === 200){
          toast.success(data.message)
        }
      })
      .catch(err => toast.error(err));
  foldername.value = '';
  props.refreshEntryInfo(props.rootpath);
}

</script>

<template>
  <Dialog v-model:open="isOpen">
    <form>
      <DialogTrigger as-child>
        <Button variant="outline">
          <FolderPlus/>new folder
        </Button>
      </DialogTrigger>
      <DialogContent class="sm:max-w-[425px]">
        <DialogHeader>
          <DialogTitle>Create Folder</DialogTitle>
          <DialogDescription>
            Create a new folder in the current location.
          </DialogDescription>
        </DialogHeader>
        <div class="grid gap-4">
          <div class="grid gap-3">
            <Label for="fname">folder name</Label>
            <Input id="fname" v-model="foldername" default-value="" placeholder="please input folder name" />
          </div>
        </div>
        <DialogFooter>
          <DialogClose as-child>
            <Button variant="outline" >
              Cancel
            </Button>
          </DialogClose>
          <Button type="submit" @click="handleSubmitButton">
            Save
          </Button>
        </DialogFooter>
      </DialogContent>
    </form>
  </Dialog>
</template>
