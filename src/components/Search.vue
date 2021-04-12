<template>
  <div class="search">
    <input @keyup="$emit('input',$event.target.value)" v-model="input" type="text" name="search" placeholder="Search Games....."/>
  </div>
</template>

<script>
export default {
  name: "Search",
  props: ["games"],
  data() {
    return {
      input: "",
    };
  },

};
</script>

<style lang="scss" scoped>
.search {
  input {
    width: calc(100% - 600px);
    position: relative;
    left: 50%;
    transform: translate(-50%);
    top: 50px;
    height: 60px;
    outline: none;
    border: 1px rgba($color: #000000, $alpha: 0.3) solid;
    box-sizing: border-box;
    padding: 0 20px;
    background: rgba($color: #fff, $alpha: 0.5);
    font-size: 18px;
    transition: ease 0.3s;

    &:focus {
      background: rgba($color: #fff, $alpha: 0.8);
    }
  }
}
</style>