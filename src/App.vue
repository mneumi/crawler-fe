<template>
  <div class="container">
    <div class="main">
      <n-card
        v-for="item in list"
        :key="item.id"
        :title="item.name"
        :bordered="false"
        :header-style="{ background: '#f2f2f2' }"
        :content-style="{ background: '#f2f2f2' }"
        hoverable
      >
        <template #cover>
          <img :src="item.image_url" />
        </template>
        {{ item.price }} 万
      </n-card>
    </div>
    <n-button type="info" v-if="hasMore" @click="getMore">查看更多</n-button>
  </div>
  <n-modal
    v-model:show="showModal"
    preset="dialog"
    title="温馨提示"
    content="此网页仅用于展示和学习使用，数据来源于爱卡汽车(xcar.com) (推荐使用电脑浏览器访问)"
    positive-text="确认"
  />
</template>

<script lang="ts">
import { defineComponent, reactive, ref, Ref } from "vue";
import { NCard, NButton, NModal } from "naive-ui";
import axios from "axios";

interface CarInfo {
  id: number;
  image_url: string;
  name: string;
  price: string;
}

const useFetchData = async (page: number) => {
  const { data } = await axios.get("http://crawler-api.mneumi.cn/api/v1/car", {
    params: {
      page,
      page_size: 20,
    },
  });

  const { list, pager } = data;
  const { page: currentPage, page_size, total_rows } = pager;

  return {
    list,
    hasMore: currentPage * page_size < total_rows,
  };
};

const useGetMore = (
  page: Ref<number>,
  list: CarInfo[],
  hasMore: Ref<boolean>
) => {
  page.value++;

  useFetchData(page.value).then((resp) => {
    list.push(...resp.list);
    hasMore.value = resp.hasMore;
  });
};

export default defineComponent({
  name: "App",
  components: {
    NCard,
    NButton,
    NModal
  },
  setup() {
    const page = ref<number>(1);
    const hasMore = ref<boolean>(true);
    const list = reactive<CarInfo[]>([]);
    const showModal = ref<boolean>(true);

    useFetchData(page.value).then((resp) => {
      list.push(...resp.list);
      hasMore.value = resp.hasMore;
    });

    const getMore = () => {
      useGetMore(page, list, hasMore);
    };

    return {
      list,
      hasMore,
      showModal,
      getMore,
    };
  },
});
</script>

<style lang="scss">
body {
  width: 100%;
  min-height: 100vh;
  background-color: green;
}

.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: green;

  .main {
    display: flex;
    justify-content: space-around;
    flex-wrap: wrap;
    width: 100%;

    .n-card {
      width: 18%;
      margin: 1vw 0;
    }
  }

  .n-button {
    margin: 10px 0;
  }
}
</style>
