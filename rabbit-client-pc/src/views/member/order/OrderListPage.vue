<template>
  <AppMemberLayout>
    <div class="member-order">
      <XtxTabs v-model:current="current">
        <XtxTabTitle v-for="item in orderStatus" :key="item.name">{{
          item.label
        }}</XtxTabTitle>
      </XtxTabs>
      <div class="order-list">
        <div class="loading" v-if="loading"></div>
        <template v-if="!loading && orderList && orderList.items.length > 0">
          <OrderItem
            :order="order"
            v-for="order in orderList.items"
            @onCancelButtonHandler="onCancelOrder"
            @onReloadOrderList="getData"
            @onLookLogistics="onLookLogistics"
            :key="order.id"
          />
        </template>
        <div v-if="!loading && orderList?.items.length === 0" class="none">
          暂无数据
        </div>
        <XtxPagination
          v-if="
            !loading & (pages > 1) && orderList && orderList.items.length > 0
          "
          v-model:page="reqParams.page"
          :pageSize="reqParams.pageSize"
          :counts="counts"
        />
      </div>
    </div>
  </AppMemberLayout>
  <CancelOrder ref="cancelOrderInstance" @onReloadOrderList="getData" />
  <OrderLogistics ref="orderLogisticsInstance" />
</template>

<script>
import { ref, watch } from "vue";
import AppMemberLayout from "@/components/AppMemberLayout";
import XtxTabs from "@/components/library/XtxTabs";
import { orderStatus } from "@/api/constants";
import { getOrderList } from "@/api/member";
import XtxTabTitle from "@/components/library/XtxTabTitle";
import OrderItem from "@/views/member/order/components/OrderItem";
import XtxPagination from "@/components/library/XtxPagination";
import CancelOrder from "@/views/member/order/components/CancelOrder";
import OrderLogistics from "@/views/member/order/components/OrderLogistics";
export default {
  name: "OrderListPage.vue",
  components: {
    OrderLogistics,
    CancelOrder,
    XtxPagination,
    OrderItem,
    XtxTabTitle,
    XtxTabs,
    AppMemberLayout,
  },
  setup() {
    const current = ref(0);
    const { orderList, reqParams, loading, counts, pages, getData } =
      useOrderList();
    watch(current, () => {
      reqParams.value.orderState = current.value;
    });
    const cancelOrderInstance = ref();
    //取消订单
    const onCancelOrder = (id) => {
      cancelOrderInstance.value.visible = true;
      cancelOrderInstance.value.id = id;
    };

    const orderLogisticsInstance = ref();

    //查看物流
    const onLookLogistics = (id) => {
      orderLogisticsInstance.value.visible = true;
      orderLogisticsInstance.value.id = id;
    };
    return {
      current,
      orderStatus,
      orderList,
      loading,
      counts,
      pages,
      reqParams,
      onCancelOrder,
      cancelOrderInstance,
      getData,
      orderLogisticsInstance,
      onLookLogistics,
    };
  },
};

function useOrderList() {
  const orderList = ref();
  const reqParams = ref({
    page: 1,
    pageSize: 3,
    orderState: 0,
  });
  //加载状态
  const loading = ref(false);
  //用于存储总数据条数
  const counts = ref(0);
  //用于存储总页数
  const pages = ref(0);
  const getData = () => {
    loading.value = true;
    getOrderList(reqParams.value).then((data) => {
      orderList.value = data.result;
      counts.value = data.result.counts;
      pages.value = data.result.pages;
      loading.value = false;
    });
  };
  //监听查询条件的变化，重新获取订单列表
  watch(reqParams.value, () => getData(), { immediate: true });
  return { orderList, reqParams, loading, counts, pages, getData };
}
</script>

<style lang="less" scoped>
.member-order {
  height: 100%;
  background: #fff;
  position: relative;
}
.order-list {
  background: #fff;
  padding: 20px;
  position: relative;
}
.loading {
  height: calc(100% - 60px);
  width: 100%;
  min-height: 400px;
  position: absolute;
  left: 0;
  top: 60px;
  background: rgba(255, 255, 255, 0.9) url(../../../assets/images/loading.gif)
    no-repeat center 18%;
}
.none {
  height: 400px;
  text-align: center;
  line-height: 400px;
  color: #999;
  background: rgba(255, 255, 255, 0.9);
}
</style>
