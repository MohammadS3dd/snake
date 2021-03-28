<template>
  <div class=" bg-indigo-100 h-screen w-screen pt-40">
    <div class="flex flex-col justify-center items-center">
      <div class="flex pointer-events-none">
        <div class="w-full"></div>
        <div
          class="inline-grid w-full"
          :style="'grid-template-columns: repeat(' + canvas.w + ', 1fr);'"
        >
          <div v-for="item in board" :key="item.id">
            <node :status="item.status" />
          </div>
        </div>
        <div class="w-full"></div>
      </div>
    </div>
    <div class="flex justify-center w-screen mt-10">
      <div class="grid grid-cols-3">
        <div class="w-20 h-20"></div>
        <button
          class="w-20 h-20 rounded-lg bg-indigo-500 text-white flex items-center justify-center focus:outline-none"
          @click="states.keys.push('w')"
        >
          up
        </button>
        <div class="w-20 h-20"></div>
        <button
          class="w-20 h-20 rounded-lg bg-indigo-500 text-white flex items-center justify-center focus:outline-none"
          @click="states.keys.push('a')"
        >
          left
        </button>
        <div class="w-20 h-20"></div>
        <button
          class="w-20 h-20 rounded-lg bg-indigo-500 text-white flex items-center justify-center focus:outline-none"
          @click="states.keys.push('d')"
        >
          right
        </button>
        <div class="w-20 h-20"></div>
        <button
          class="w-20 h-20 rounded-lg bg-indigo-500 text-white flex items-center justify-center focus:outline-none"
          @click="states.keys.push('s')"
        >
          down
        </button>
        <div class="w-20 h-20"></div>
      </div>
    </div>
    <div class="text-indigo-700 my-4 font-bold font-sans">you can also use W,A,S,D to move</div>
    <h1 class="text-indigo-700 font-bold font-sans" v-if="states.state == 'lost'">you lost . press f5 to continue</h1>
    <label class="text-indigo-700" >
      speed: 
      <input v-model="states.speed" class="focus:outline-none" type="range" min="1" max="100" name="" id="">
    </label>
  </div>
</template>

<script>
import node from "./node.vue";
import { onMounted, reactive } from "vue";
export default {
  components: {
    node,
  },

  setup() {
    console.log(
      "/// this is just a simple project to test Vue's performance  ///"
    );
    console.log("/// take it with a grain of salt  ///");
    const states = reactive({
      frameinterval: 1000,

      state: "running",
      food: [],
      keys: [],
      frame: 0,
      timing: 0,
      speed:10
    });
    const canvas = { w: 90, h: 30 };
    const b = [
      { status: "off", id: 0 },
      { status: "start", id: 1 },
      { status: "end", id: 2 },
    ];

    for (let index = 0; index < canvas.w * canvas.h; index++) {
      b[index] = { status: "off", id: index };
      //   index % canvas.w == 0
      //     ? (b[index].status = "start")
      //     : (b[index].status = "off");
    }
    const toindex = function (i, j) {
      return canvas.w * i + j;
    };
    const toCOORDS = function (i) {
      return [parseInt(i / canvas.w), i % canvas.w];
    };

    b[toindex(7, 2)].status = "snake";
    const board = reactive(b);
    const change = function (i, j, val) {
      board[toindex(i, j)].status = val;
    };
    const changeFI = function (i, val) {
      board[i].status = val;
    };
    const snake = { body: [75], dir: "l" };


    const next = function () {
      if (states.frame == 1) {
        snake.body.push(snake.body[0]);
        snake.body.push(snake.body[0]);
      }
      if (states.state == "lost") {
        return;
      }
      if(states.frame > 2**16 -10){
        states.frame = 10
      }
      states.frame++;
      states.frameinterval = 1000 / states.speed
      const head = snake.body[0];

      // change snakes direction
      const key = states.keys[0];
      key == "w" && snake.dir != "d"
        ? (snake.dir = "u")
        : key == "a" && snake.dir != "r"
        ? (snake.dir = "l")
        : key == "s" && snake.dir != "u"
        ? (snake.dir = "d")
        : key == "d" && snake.dir != "l"
        ? (snake.dir = "r")
        : null;

      // move snake

      const span = canvas.w * canvas.h;
      const movement = {
        up: () => {
          snake.body.unshift((((head - canvas.w) % span) + span) % span);
          snake.body.pop();
        },
        down: () => {
          snake.body.unshift((((head + canvas.w) % span) + span) % span);
          snake.body.pop();
        },
        left: () => {
          const cordsP = toCOORDS(head);
          const cordsN = toCOORDS(head - 1 + canvas.w);

          snake.body.unshift(toindex(cordsP[0], cordsN[1]));
          snake.body.pop();
        },
        right: () => {
          const cordsP = toCOORDS(head);
          const cordsN = toCOORDS(head + 1);

          snake.body.unshift(toindex(cordsP[0], cordsN[1]));
          snake.body.pop();
        },
      };

      snake.dir == "u"
        ? movement.up()
        : snake.dir == "d"
        ? movement.down()
        : snake.dir == "l"
        ? movement.left()
        : snake.dir == "r"
        ? movement.right()
        : null;
      // undraw snake
      board.forEach((element) => {
        element.status == "snake" ? (element.status = "off") : null;
      });

      //draw snake
      snake.body.forEach((element) => {
        changeFI(element, "snake");
      });

      states.keys.shift();
      states.keys.length > 2 ? (states.keys = []) : null;
      states.food.forEach((element, index) => {
        if (element == snake.body[0]) {
          states.food.splice(index, 1);
          snake.body.push(head);
          addFood();
        }
      });
      for (let index = 1; index < snake.body.length; index++) {
        const element = snake.body[index];
        if (element == snake.body[0]) {
          states.frameinterval = 1000000;
          states.state = "lost";
        }
      }
    };
    const addFood = function () {
      const rand = parseInt(Math.random() * canvas.w * canvas.h - 1);
      states.food.push(rand);

      board[rand] = { id: rand, status: "food" };
    };

    const keypressed = function (e) {
      const key = String.fromCharCode(e.keyCode);
      states.keys.push(key);
    };

    const draw = function (t) {
      if (states.timing === 0 || !states.timing) {
        states.timing = t;
      }
      if (t - states.timing > states.frameinterval) {
        next();
        states.timing = t;
      }
      requestAnimationFrame(draw);
    };

    onMounted(() => {
      document.addEventListener("keypress", keypressed);
      addFood();
      draw();
    });

    return {
      board,
      canvas,
      toindex,
      change,
      snake,
      next,
      addFood,
      states,
      keypressed,
    };
  },
};
</script>

<style>
</style>
