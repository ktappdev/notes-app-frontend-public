<template>
  <div class="instructions">
    <div>
      <h1>Secret Note</h1><br>
      <b>Now you can add or replace an image with every note!</b>
      <br />
      Notes and Images stored on "secretnote" can be public or private,
      depending on the title. Simple titles lead to highly visible notes that
      can be edited by anyone who stumbles upon them.<br />
      <br />
      Using common words like "hello" as the title is for entertainment purposes
      only. You will find a variety of messages left by others. If you find
      something offensive, delete it.<br />
      All data is stores in a database with AES-128bit encription and only your
      title can decrypt it.
      <br />
      <p class="footer">
        Copyright &copy; ktad 2022
      </p><br />
    </div>
  </div>
</template>

<script>
export default {
  name: `Instructions`,
};
</script>

<style scoped>
.instructions {
  font-family: Courier New,Courier,monospace;
  position: fixed;
  left: 15%;
  right: 15%;
  font-size: medium;
}
.footer{
  font-size: x-small;
}
</style>
