<template>
  <div class="w-2 h-2 " :class="status"></div>
</template>

<script>
export default {
  props: { status },
};
</script>

<style>

.off {
  background-color: aliceblue;
}
.start {
  background-color: rgb(124, 157, 226);
}
.end {
  background-color: cadetblue;
}
.ongoing {
  background-color: rgb(224, 231, 160);
}
.snake {
  background-color: aquamarine;
}
.food {
  background-color: darkred;
}
</style>
