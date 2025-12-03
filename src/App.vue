<script setup>
import Navbar from './components/navbar.vue'
import { ref, onMounted } from 'vue'
import { supabase } from '@/lib/supabaseClient.js'

console.log('Test Variable:', import.meta.env.VITE_TEST_VAR);
console.log("Supabase URL?", import.meta.env.VITE_SUPABASE_URL)

</script>

<template>
  <navbar></navbar>
  <router-view></router-view> 
</template>