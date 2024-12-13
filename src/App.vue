<template>
  <h1>Spinning Wheel🍩!</h1>
  <section>
    <div class="wheel">
      <SpinningWheel :segments="wheelSegments" :preSelected="desiredPrize" />
    </div>
    <div class="panel">
      <h2>Edit Gifts🎄 Here</h2>
      <ul>
        <li
          v-for="(segment, index) in wheelSegments"
          :key="index"
          :class="{ selected: index === selectedIndex, selecting: selectMode }"
          @click="selectSegment(index)"
        >
          <input
            type="text"
            v-model="wheelSegments[index]"
            :readonly="selectMode"
          />
          <span class="delete-icon" @click.stop="deleteSegment(index)">❌</span>
        </li>
      </ul>
      <button @click="toggleSelectMode">Make a wish</button>
      <p>
        Wishing for ...
        <template v-if="selectedIndex">
          <b>{{ wheelSegments[selectedIndex] }} !</b>
          <p>Let's Spin again!</p>
        </template>
      </p>
    </div>
  </section>

  <AppFooter />
</template>

<script>
import AppFooter from "./components/AppFooter.vue";
import SpinningWheel from "./components/SpinningWheel.vue";
import { ref } from "vue";

export default {
  name: "App",
  components: {
    SpinningWheel,
    AppFooter,
  },
  setup() {
    const wheelSegments = ref([
      "Candy Cane",
      "Xbox Series X",
      "PlayStation 5",
      "Holiday Sweater",
      "Chocolate Santa",
      "Apple TV 4K",
      "Christmas Tree",
      "Nintendo Switch",
    ]);
    const desiredPrize = ref(null);
    const selectMode = ref(false); // 是否处于选择模式
    const selectedIndex = ref(null); // 当前选中的li的索引

    const deleteSegment = (i) => {
      wheelSegments.value.splice(i, 1);
      // 如果当前选中的被删除，则重置选中状态
      if (selectedIndex.value === i) {
        selectedIndex.value = null;
      }
    };

    const toggleSelectMode = () => {
      selectMode.value = !selectMode.value;
      // 清空已选中的状态
      if (!selectMode.value) {
        selectedIndex.value = null;
      }
    };

    const selectSegment = (i) => {
      // 只有在选择模式下才能选择
      if (selectMode.value) {
        selectedIndex.value = i;
        desiredPrize.value = wheelSegments.value[i];
        selectMode.value = false;
      }
    };

    return {
      wheelSegments,
      desiredPrize,
      selectMode,
      selectedIndex,
      deleteSegment,
      selectSegment,
      toggleSelectMode,
    };
  },
};
</script>

<style>
#app {
  display: flex;
  flex-direction: column;
  justify-content: space-between;

  height: 100vh;

  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

h1 {
  font-size: 32px;
}

.selecting {
  opacity: 0.8;

  input {
    cursor: pointer;
  }

  span {
    display: none;
  }
}

.selecting:hover {
  opacity: 1;
  box-shadow: rgba(0, 0, 0, 0.16) 0px 1px 4px;
}

.selected {
  background: white;
  box-shadow: rgba(0, 0, 0, 0.16) 0px 1px 4px;
}

ul {
  display: flex;
  flex-direction: column;
  gap: 6px;

  margin: 0 auto;
  padding: 12px;

  width: fit-content;

  background: #f0f0f0;
  border-radius: 12px;

  li {
    position: relative;
    list-style: none;
    border-radius: 6px;

    input {
      padding: 6px 12px 6px 42px;

      background: rgba(255, 255, 255, 0.9);
      font-size: 18px;
      border-radius: 6px;
      border: none;
    }

    input:focus {
      outline: none;
    }

    .delete-icon {
      position: absolute;
      right: 12px;
      top: 15%;

      margin-left: 6px;

      width: 24px;
      height: 24px;

      font-size: 14px;
      line-height: 24px;

      background: white;

      cursor: pointer;
      opacity: 0.5;
    }
  }

  li:hover {
    .delete-icon {
      opacity: 1;
    }
  }
}

section {
  display: flex;
  margin: 0 120px;

  > div {
    flex: 1;
  }
}

.panel {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;

  button {
    padding: 12px 24px;

    width: fit-content;

    cursor: pointer;

    border: none;
    border-radius: 12px;
    font-size: 18px;
  }
}
</style>
