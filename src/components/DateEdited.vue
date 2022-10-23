<template>
  <div class="dateEdited">
    <p id="dateEditedId">
      {{date_}}
    </p>
  </div>
</template>

<script>
export default {
  name: "DateEdited",
  props: {
  date_: String
  },
}
</script>

<style scoped>
#dateEditedId{
  opacity: 40%;

}
</style>