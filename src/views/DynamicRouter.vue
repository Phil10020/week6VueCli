<template>
    <h1>
        router
    </h1>
</template>

<script>
import axios from 'axios'
export default {
  created () {
    console.log(this.$route.params.id)
    axios.get('https://randomuser.me/api/')
      .then((res) => console.log(res))
  }
}
</script>
