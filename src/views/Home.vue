<template>
  <div class="home">
    <Calendar />
  </div>
</template>

<script>
// @ is an alias to /src
import { Calendar } from '@/components';

export default {
  name: 'home',
  components: {
    Calendar,
  },
};
</script>
