<template>
  <div class="wheel-container">
    <div class="wheel-wrapper">
      <!-- 固定的箭头 -->
      <div class="pointer"></div>

      <canvas
        ref="wheelCanvas"
        :width="canvasSize"
        :height="canvasSize"
        :class="{ spinning: isSpinning }"
      ></canvas>
      <div
        class="center-button"
        @click="spin"
        :class="{ spinning: isSpinning }"
      >
        Spin
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, watch } from "vue";

export default {
  name: "SpinningWheel",
  props: {
    segments: {
      type: Array,
      required: true,
    },
    canvasSize: {
      type: Number,
      default: 500,
    },
    spinDuration: {
      type: Number,
      default: 5000, // 旋转持续时间（毫秒）
    },
    preSelected: {
      type: String,
      default: null, // 预定选中的奖品
    },
  },
  setup(props) {
    let ctx = null;
    const wheelCanvas = ref(null);
    const isSpinning = ref(false);
    const selectedSegment = ref(null);
    const center = props.canvasSize / 2;
    const radius = center - 10; // 边距
    const anglePerSeg = 360 / props.segments.length;

    // 绘制轮盘
    const drawWheel = () => {
      const segCount = props.segments.length;
      const anglePerSegmentRad = (2 * Math.PI) / segCount;
      ctx.clearRect(0, 0, props.canvasSize, props.canvasSize);

      // 绘制每个段落
      props.segments.forEach((segment, index) => {
        const startAngle = index * anglePerSegmentRad - Math.PI / 2; // 旋转-90度，使第一个段落位于顶部
        const endAngle = startAngle + anglePerSegmentRad;

        const colors = ["#F2E8CF", "#BC4749"];
        ctx.fillStyle = colors[index % colors.length];

        ctx.beginPath();
        ctx.moveTo(center, center);
        ctx.arc(center, center, radius, startAngle, endAngle);
        ctx.closePath();
        ctx.fill();

        // 绘制段落文本
        ctx.save();
        ctx.translate(center, center);
        ctx.rotate(startAngle + anglePerSegmentRad / 2);
        ctx.textAlign = "right";
        const fontColors = ["#BC4749", "#F2E8CF"];
        ctx.fillStyle = fontColors[index % colors.length];
        ctx.font = "bold 18px Arial";
        ctx.fillText(segment, radius - 10, 0);
        ctx.restore();
      });
    };

    // 初始化画布并绘制轮盘
    onMounted(() => {
      ctx = wheelCanvas.value.getContext("2d");
      drawWheel();
    });

    // 监听segments变化，重新绘制轮盘
    watch(
      () => props.segments,
      (newSegments) => {
        if (props.preSelected && newSegments.includes(props.preSelected)) {
          selectedSegment.value = props.preSelected;
        } else {
          selectedSegment.value = null;
        }
        drawWheel();
      },
      { deep: true }
    );

    watch(
      () => props.preSelected,
      (newVal) => {
        if (newVal && props.segments.includes(newVal)) {
          selectedSegment.value = newVal;
        } else {
          selectedSegment.value = null;
        }
        drawWheel();
      }
    );

    // 旋转函数
    const spin = () => {
      if (isSpinning.value) return;

      // 确定选中的奖品
      let targetSegment = null;
      if (props.preSelected && props.segments.includes(props.preSelected)) {
        targetSegment = props.preSelected;
      } else {
        // 如果没有预定选中的奖品，则随机选择
        const randomIndex = Math.floor(Math.random() * props.segments.length);
        targetSegment = props.segments[randomIndex];
      }

      isSpinning.value = true;
      selectedSegment.value = null;

      // 找到目标段落的索引
      const targetIndex = props.segments.indexOf(targetSegment);
      const targetAngle = targetIndex * anglePerSeg + anglePerSeg / 2;

      // 计算目标旋转角度，使得 targetAngle 位于指针下方（0度）
      const additionalSpins = 5; // 旋转的圈数
      const totalRotation = 360 * additionalSpins + (360 - targetAngle);

      // 应用CSS旋转
      wheelCanvas.value.style.transition = `transform ${props.spinDuration}ms ease-out`;
      wheelCanvas.value.style.transform = `rotate(${totalRotation}deg)`;

      // 旋转结束后的处理
      setTimeout(() => {
        isSpinning.value = false;

        wheelCanvas.value.style.transition = "none";

        const actualRotation = totalRotation % 360;
        // const selectedIndex = Math.floor((360 - actualRotation) / anglePerSeg) % segCount;

        selectedSegment.value = targetSegment;
        alert(`Got ${selectedSegment.value} !`);

        // 重置旋转以防止旋转角度过大
        wheelCanvas.value.style.transform = `rotate(${actualRotation}deg)`;
      }, props.spinDuration);
    };

    return {
      wheelCanvas,
      spin,
      isSpinning,
      selectedSegment,
    };
  },
};
</script>

<style scoped>
.wheel-container {
  display: flex;
  flex-direction: column;
  align-items: center;

  padding: 24px;
}

.wheel-wrapper {
  position: relative;
  width: 500px;
  height: 500px;
}

.pointer {
  position: absolute;
  top: 10px;
  left: 50%;

  width: 0;
  height: 0;

  border-left: 15px solid transparent;
  border-right: 15px solid transparent;
  border-top: 20px solid white;

  z-index: 3;
  transform: translateX(-50%);
}

canvas {
  border: 2px dashed rgba(128, 128, 128, 0.5);
  border-radius: 50%;
  transition: transform 5s ease-out;
  box-shadow: rgba(0, 0, 0, 0.16) 0px 1px 4px;
}

.center-button {
  position: absolute;
  top: 50%;
  left: 50%;

  display: flex;
  align-items: center;
  justify-content: center;

  width: 80px;
  height: 80px;

  background-color: #fff;
  box-shadow: rgba(0, 0, 0, 0.16) 0px 1px 4px;
  border-radius: 50%;

  cursor: pointer;
  user-select: none;
  font-size: 20px;
  font-weight: bold;
  transform: translate(-50%, -50%);
  transition: background-color 0.3s, opacity 0.3s;
}

.center-button:hover {
  background-color: #f0f0f0;
}

.center-button.spinning {
  cursor: not-allowed;
}
</style>
