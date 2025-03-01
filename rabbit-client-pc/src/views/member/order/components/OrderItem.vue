<template>
  <div class="order-item">
    <div class="head">
      <span>下单时间：{{ order.createTime }}</span>
      <span>订单编号：{{ order.id }}</span>
      <span
        class="down-time"
        v-if="order.orderState === 1 && order.countdown !== -1"
      >
        <i class="iconfont icon-down-time"></i>
        <b>付款截止：{{ dayjs.unix(count).format("mm分ss秒") }}</b>
      </span>
      <a
        @click="deleteOrderHandler(order.id)"
        v-if="[5, 6].includes(order.orderState)"
        href="javascript:"
        class="del"
        >删除</a
      >
    </div>
    <div class="body">
      <div class="column goods">
        <ul>
          <li v-for="goods in order.skus" :key="goods.id">
            <RouterLink class="image" :to="`/goods/${goods.id}`">
              <img :src="goods.image" alt="" />
            </RouterLink>
            <div class="info">
              <p class="name ellipsis-2">{{ goods.name }}</p>
              <p class="attr ellipsis">
                <span>{{ goods.attrsText }}</span>
              </p>
            </div>
            <div class="price">¥{{ goods.realPay }}</div>
            <div class="count">x{{ goods.quantity }}</div>
          </li>
        </ul>
      </div>
      <div class="column state">
        <!-- orderStatus是一个数组, 数组的索引和订单状态值是对应关系 -->
        <p>{{ orderStatus[order.orderState].label }}</p>
        <p v-if="order.orderState === 3">
          <a @click="lookLogistics(order.id)" href="javascript:" class="green"
            >查看物流</a
          >
        </p>
        <p v-if="order.orderState === 4">
          <a href="javascript:" class="green">评价商品</a>
        </p>
        <p v-if="order.orderState === 5">
          <a href="javascript:" class="green">查看评价</a>
        </p>
      </div>
      <div class="column amount">
        <p class="red">¥{{ order.payMoney }}</p>
        <p>（含运费：¥{{ order.postFee }}）</p>
        <p>在线付款</p>
      </div>
      <div class="column action">
        <XtxButton v-if="order.orderState === 1" type="primary" size="small"
          ><RouterLink :to="`/checkout/pay?orderId=${order.id}`"
            >立即付款</RouterLink
          ></XtxButton
        >
        <XtxButton
          @click="receiptGoodsHandler(order.id)"
          v-if="order.orderState === 3"
          type="primary"
          size="small"
          >确认收货</XtxButton
        >
        <p>
          <RouterLink :to="`/member/order/${order.id}`">查看详情</RouterLink>
        </p>
        <p>
          <a
            v-if="order.orderState === 1"
            @click="onCancelButtonClick(order.id)"
            href="javascript:"
            >取消订单</a
          >
        </p>
        <p>
          <a v-if="[2, 3, 4, 5].includes(order.orderState)" href="javascript:"
            >再次购买</a
          >
        </p>
        <p>
          <a v-if="[4, 5].includes(order.orderState)" href="javascript:"
            >申请售后</a
          >
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import { useCountDown } from "@/hooks/useCountDown";
import dayjs from "dayjs";
import { orderStatus } from "@/api/constants";
import Confirm from "@/components/library/confirm";
import { deleteOrder, receiptGoods } from "@/api/member";
import Message from "@/components/library/message";

export default {
  name: "OrderItem",
  props: {
    order: {
      type: Object,
      default: () => ({}),
    },
  },
  setup(props, { emit }) {
    const { count, start } = useCountDown();
    if (props.order.orderState === 1 && props.order.countdown !== -1) {
      //开始倒计时
      start(1800);
    }
    //取消订单
    const onCancelButtonClick = (id) => {
      emit("onCancelButtonHandler", id);
    };
    //删除订单
    const deleteOrderHandler = async (id) => {
      //和用户进行确认
      try {
        //和用户确认
        await Confirm({ content: "确定要删除订单吗" });
        //发起请求
        await deleteOrder([id]);
        Message({ type: "success", text: "订单删除成功" });
        //更新列表
        emit("onReloadOrderList");
      } catch (error) {
        Message({ type: "success", text: "订单删除失败" });
      }
    };
    //确认收获
    const receiptGoodsHandler = (id) => {
      Confirm({ content: "确认收货？" })
        .then(() => receiptGoods(id))
        .then(() => emit("onReloadOrderList"))
        .then(() => Message({ type: "success", text: "收货成功" }))
        .catch(() => Message({ type: "success", text: "收货失败" }));
    };

    const lookLogistics = (id) => {
      emit("onLookLogistics", id);
    };
    return {
      count,
      dayjs,
      orderStatus,
      onCancelButtonClick,
      deleteOrderHandler,
      receiptGoodsHandler,
      lookLogistics,
    };
  },
};
</script>

<style lang="less" scoped>
.order-item {
  margin-bottom: 20px;
  border: 1px solid #f5f5f5;
  .head {
    height: 50px;
    line-height: 50px;
    background: #f5f5f5;
    padding: 0 20px;
    overflow: hidden;
    span {
      margin-right: 20px;
      &.down-time {
        margin-right: 0;
        float: right;
        i {
          vertical-align: middle;
          margin-right: 3px;
        }
        b {
          vertical-align: middle;
          font-weight: normal;
        }
      }
    }
    .del {
      margin-right: 0;
      float: right;
      color: #999;
    }
  }
  .body {
    display: flex;
    align-items: stretch;
    .column {
      border-left: 1px solid #f5f5f5;
      text-align: center;
      padding: 20px;
      > p {
        padding-top: 10px;
      }
      &:first-child {
        border-left: none;
      }
      &.goods {
        flex: 1;
        padding: 0;
        align-self: center;
        ul {
          li {
            border-bottom: 1px solid #f5f5f5;
            padding: 10px;
            display: flex;
            &:last-child {
              border-bottom: none;
            }
            .image {
              width: 70px;
              height: 70px;
              border: 1px solid #f5f5f5;
            }
            .info {
              width: 220px;
              text-align: left;
              padding: 0 10px;
              p {
                margin-bottom: 5px;
                &.name {
                  height: 38px;
                }
                &.attr {
                  color: #999;
                  font-size: 12px;
                  span {
                    margin-right: 5px;
                  }
                }
              }
            }
            .price {
              width: 100px;
            }
            .count {
              width: 80px;
            }
          }
        }
      }
      &.state {
        width: 120px;
        .green {
          color: @xtxColor;
        }
      }
      &.amount {
        width: 200px;
        .red {
          color: @priceColor;
        }
      }
      &.action {
        width: 140px;
        a {
          display: block;
          &:hover {
            color: @xtxColor;
          }
        }
      }
    }
  }
}
</style>
