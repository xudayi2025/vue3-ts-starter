<template>
  <div class="list-container">
    <!-- 渲染列表项 -->
    <div
      v-for="(item, index) in list"
      :key="index"
      :style="{ backgroundColor: item.background }"
      class="list-item"
    >
      {{ index }}
    </div>

    <!-- Loading 状态 -->
    <div v-if="loading" class="loading">
      Loading...
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue';

// 定义数据类型
interface ListItem {
  background: string;
}

// 初始化基础变量
const list = ref<ListItem[]>([{ background: 'rgb(233,32,38)' }]);
const loading = ref<boolean>(false);
let stopLoading = false; // 控制是否停止加载

// 生成随机颜色
const getRandomColor = (): string => {
  const r = Math.floor(Math.random() * 256);
  const g = Math.floor(Math.random() * 256);
  const b = Math.floor(Math.random() * 256);
  return `rgb(${r},${g},${b})`;
};

// 模拟数据获取方法
const getList = async (num: number): Promise<ListItem[]> => {
  // 模拟延迟 [0.5] 秒
  await new Promise((resolve) => setTimeout(resolve, Math.random() * 500));

  // 返回 num 个随机颜色的数组
  const result: ListItem[] = [];
  for (let i = 0; i < num; i++) {
    result.push({ background: getRandomColor() });
  }
  return result;
};

// 滚动事件处理函数
const handleScroll = async () => {
  if (loading.value || stopLoading) return;

  // 检查是否滚动超过一半
  const scrollTop = window.scrollY || document.documentElement.scrollTop;
  const clientHeight = document.documentElement.clientHeight;
  const scrollHeight = document.documentElement.scrollHeight;

  // 当滚动位置超过页面高度的一半时，继续加载
  if (scrollTop + clientHeight >= scrollHeight / 2) {
    loading.value = true;

    try {
      const newData = await getList(10);
      list.value = [...list.value, ...newData];

      // 如果渲染超过 50 个则停止
      if (list.value.length >= 50) {
        stopLoading = true;
      }
    } catch (error) {
      console.error('Failed to load data:', error);
    } finally {
      loading.value = false;
    }
  }
};

// 组件初始化后调用 getList(10)
onMounted(async () => {
  // 初始加载数据
  loading.value = true;
  try {
    const initialData = await getList(10);
    list.value = [...list.value, ...initialData];
  } catch (error) {
    console.error('Failed to load initial data:', error);
  } finally {
    loading.value = false;
  }

  // 监听滚动事件
  window.addEventListener('scroll', handleScroll);
});

// 组件卸载前移除事件监听
onBeforeUnmount(() => {
  window.removeEventListener('scroll', handleScroll);
});
</script>

<style scoped>
.list-container {
  width: 100%;
}

.list-item {
  width: 100%;
  height: 500px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 48px;
  color: white;
  font-weight: bold;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
}

.loading {
  width: 100%;
  padding: 20px;
  text-align: center;
  font-size: 24px;
  color: #666;
  background-color: #f0f0f0;
}
</style>
