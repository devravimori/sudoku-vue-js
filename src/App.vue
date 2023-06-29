<script setup>
import { reactive, ref } from "@vue/reactivity";
import { useToast } from "vue-toastification";
import { makepuzzle, solvepuzzle, ratepuzzle } from "sudoku";
const toast = useToast();
const inputValue = ref();
let cell = [...document.querySelectorAll(".cell")];
let puzzleOld = null;
const sudoku = reactive({
  puzzle: Array([]),
  solution: Array([]),
  Answer: Array([]),
  showSolution: false,
});

const getPuzzle = () => {
  let p = makepuzzle();
  let s = solvepuzzle(p);
  let c = 0;
  p.map((su, index) => {
    if (su === 0) su = 9;
    if (su === null) su = 0;
    sudoku.puzzle[c].push(su);
    if ((index + 1) % 9 === 0 && index + 1 != 81) {
      c++;
      sudoku.puzzle[c] = [];
    }
  });
  c = 0;

  s.map((su, index) => {
    if (su === 0) su = 9;
    sudoku.solution[c].push(su);
    if ((index + 1) % 9 === 0 && index + 1 != 81) {
      c++;
      sudoku.solution[c] = [];
    }
  });
  console.log(sudoku.solution);
  c = 0;
};

getPuzzle();

const changePuzzle = () => {
  cell.map((c) => {
    c.classList.remove("bg-red-100");
  });
  sudoku.puzzle = Array([]);
  sudoku.solution = Array([]);
  sudoku.Answer = Array([]);
  getPuzzle();
};

const box = (row, col) => {
  let className = "";
  if (col == 3 || col == 6) className += " border-r-[2px]";
  if (row == 3 || row == 6) className += " border-b-[2px]";
  // if (
  //   (col > 3 && col < 7 && (row < 4 || row > 6)) ||
  //   (row > 3 && row < 7 && (col < 4 || col > 6))
  // )
  //   className += " bg-green-100";
  return className;
};

const nextInput = (e, row, col) => {
  if (!e.ctrlKey && !e.altKey && e.key > 0 && e.key < 10) {
    if (e.srcElement.value.length > 1) {
      e.srcElement.value = e.srcElement.value.slice(-1);
    }

    if (e.srcElement.value.length > 0) {
      let next = "";
      if (col === 9 && row !== 9) {
        next = `${row + 1}-1`;
      } else if (row == 9 && col === 9) {
        next = `1-1`;
      } else {
        next = `${row}-${col + 1}`;
      }
      hasNextReadonly(next);
    }
  } else {
    e.srcElement.value = "";
  }
};

const hasNextReadonly = (next) => {
  let el = document.querySelector(`[data-value="${next}"]`);
  if (el.hasAttribute("readonly")) {
    if (
      parseInt(next.split("-")[0]) !== 9 &&
      parseInt(next.split("-")[1]) === 9
    ) {
      return hasNextReadonly(`${parseInt(next.split("-")[0]) + 1}-1`);
    } else if (
      parseInt(next.split("-")[0]) === 9 &&
      parseInt(next.split("-")[1]) === 9
    ) {
      return hasNextReadonly(`1-1`);
    } else {
      return hasNextReadonly(
        `${next.split("-")[0]}-${parseInt(next.split("-")[1]) + 1}`
      );
    }
  }

  el.focus();
};

const checkAnswer = () => {
  cell.map((c) => {
    c.classList.remove("bg-red-100");
  });
  sudoku.Answer = Array([]);
  let element = inputValue._rawValue;
  let c = 0;
  for (let index = 0; index < element.length; index++) {
    const i = element[index];
    if (i.value == "") {
      toast.error("Some filed is empty", {
        timeout: 2000,
      });
      return 0;
    }

    sudoku.Answer[c].push(parseInt(i.value));
    if ((index + 1) % 9 === 0 && index + 1 !== 81) {
      c++;
      sudoku.Answer[c] = [];
    }
  }
  compareAnswer();
};

const compareAnswer = () => {
  let error = false;
  for (let i = 0; i < sudoku.solution.length; i++) {
    for (let j = 0; j < sudoku.Answer.length; j++) {
      if (sudoku.solution[i][j] !== sudoku.Answer[i][j]) {
        console.log(sudoku.solution[i][j]);
        let el = document.querySelector(`[data-value="${i + 1}-${j + 1}"]`);
        if (!error) {
          cell.map((c) => {
            c.classList.remove("bg-red-100");
          });
          error = true;
          toast.error("Wrong answer", {
            timeout: 2000,
          });
        }

        el.parentElement.classList.add("bg-red-100"),
          `[data-value="${i}-${j}"]`;

        document
          .querySelector(`[data-value="${i + 1}-${j + 1}"]`)
          .parentElement.classList.add("bg-red-100");
      }
    }
  }
  if (!error) {
    toast.success("You did it. correct answer", {
      timeout: 2000,
    });
  }
};

const puzzleSolution = () => {
  if (!sudoku.showSolution) {
    puzzleOld = sudoku.puzzle;
    sudoku.puzzle = sudoku.solution;
    sudoku.showSolution = true;
  } else {
    sudoku.puzzle = puzzleOld;
    sudoku.showSolution = false;
  }
};

const selectAllText = (e) => {
  e.srcElement.select();
};
</script>

<template>
  <div className="bg-[#e6f0f0] h-screen ">
    <div class="max-w-4xl mx-auto sm:px-6 lg:px-8">
      <div class="flex flex-col items-center justify-center h-screen">
        <div
          class="
            p-8
            pb-3
            bg-white
            overflow-hidden
            shadow
            sm:rounded-lg
            text-[#566b6b]
          "
        >
          <div v-for="row in 9" class="flex" :key="row">
            <div
              v-for="col in 9"
              :key="col"
              class="
                border border-gray-400
                p-2
                pt-1
                lg:pt-2
                h-8
                w-8
                lg:w-10 lg:h-10
                cell
              "
              :class="[
                box(row, col),
                sudoku.puzzle[row - 1][col - 1] !== 0 && 'bg-green-100',
              ]"
            >
              <input
                ref="inputValue"
                :data-value="row + '-' + col"
                type="number"
                class="
                  w-3
                  lg:w-5
                  focus:outline-none
                  text-center
                  bg-transparent
                  text-gray-600
                  font-semibold
                  text-sm
                "
                :value="
                  sudoku.puzzle[row - 1][col - 1] === 0
                    ? ''
                    : sudoku.puzzle[row - 1][col - 1]
                "
                @keyup="nextInput($event, row, col)"
                @focus="selectAllText($event)"
                autocomplete="off"
                :readonly="sudoku.puzzle[row - 1][col - 1] !== 0"
              />
            </div>
          </div>
          <div class="flex justify-between mt-2">
            <div class="flex space-x-4">
              <!-- <button class="text-[#00dc82]">Answer</button> -->
              <button class="text-[#00dc82]" @click="changePuzzle">
                Change
              </button>
              <button class="text-[#00dc82]" @click="puzzleSolution">
                <span v-if="!sudoku.showSolution">Show Solution</span>
                <span v-else>Hide Solution</span>
              </button>
            </div>
            <button
              class="text-[#00dc82]"
              :class="sudoku.showSolution && 'cursor-not-allowed'"
              @click="checkAnswer"
              :disabled="sudoku.showSolution"
            >
              Check
            </button>
          </div>
        </div>
        <div
          class="mt-2 flex justify-between"
          style="width: calc(4rem + 2.5rem * 9)"
        ></div>
        
        <div class="mt-5 font-semibold">Made with 💖 by <a href="https://optimumitapps.com/" class="text-[#00dc82]">Optimumitapps</a></div>
      </div>
      
    </div>
  </div>
</template>
