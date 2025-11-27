<script setup lang="ts">
import {
  NavigationMenu,
  NavigationMenuItem,
  NavigationMenuList,
  NavigationMenuLink,
} from '@/components/ui/navigation-menu'
import {Server} from "lucide-vue-next";

</script>

<template>

  <NavigationMenu :viewport="false" >
    <NavigationMenuList >
      <NavigationMenuItem >
        <NavigationMenuLink><span class="flex items-center gap-2 text-3xl font-bold"><Server class="size-10"/>RSHttpServer</span></NavigationMenuLink>
      </NavigationMenuItem>
    </NavigationMenuList>
  </NavigationMenu>

</template>
