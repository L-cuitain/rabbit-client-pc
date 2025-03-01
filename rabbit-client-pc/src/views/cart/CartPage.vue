<template>
  <AppLayout>
    <div class="cart-page">
      <div class="container">
        <XtxBread>
          <XtxBreadItem path="/">首页</XtxBreadItem>
          <XtxBreadItem>购物车</XtxBreadItem>
        </XtxBread>
        <div class="cart">
          <table>
            <thead>
              <tr>
                <th>
                  <XtxCheckbox
                    :modelValue="selectAllButtonStatus"
                    @update:modelValue="
                      $store.dispatch('cart/selectedAll', $event)
                    "
                    >全选</XtxCheckbox
                  >
                </th>
                <th>商品信息</th>
                <th>单价</th>
                <th>数量</th>
                <th>小计</th>
                <th>操作</th>
              </tr>
            </thead>
            <!-- 有效商品 -->
            <tbody>
              <tr v-if="effectiveGoodsCount === 0">
                <td colspan="6">
                  <EmptyCart />
                </td>
              </tr>
              <tr v-else v-for="item in effectiveGoodsList" :key="item.id">
                <td>
                  <XtxCheckbox
                    :modelValue="item.selected"
                    @update:modelValue="
                      $store.dispatch('cart/updateGoodsOfCartBySkuId', {
                        skuId: item.skuId,
                        selected: $event,
                      })
                    "
                  />
                </td>
                <td>
                  <div class="goods">
                    <RouterLink :to="`/goods/${item.id}`"
                      ><img :src="item.picture" alt=""
                    /></RouterLink>
                    <div>
                      <p class="name ellipsis">
                        {{ item.name }}
                      </p>
                      <!-- 选择规格组件 -->
                      <CartSku
                        :attrsText="item.attrsText"
                        :skuId="item.skuId"
                      />
                    </div>
                  </div>
                </td>
                <td class="tc">
                  <p>&yen;{{ item.nowPrice }}</p>
                  <p v-if="item.price - item.nowPrice">
                    比加入时降价
                    <span class="red"
                      >&yen;{{ (item.price - item.nowPrice).toFixed(2) }}</span
                    >
                  </p>
                </td>
                <td class="tc">
                  <XtxNumberBox
                    :max="item.stock"
                    :modelValue="item.count"
                    @update:modelValue="
                      $store.dispatch('cart/updateGoodsOfCartBySkuId', {
                        skuId: item.skuId,
                        count: $event,
                      })
                    "
                  ></XtxNumberBox>
                </td>
                <td class="tc">
                  <p class="f16 red">
                    &yen;{{ item.count * Number(item.nowPrice).toFixed(2) }}
                  </p>
                </td>
                <td class="tc">
                  <p><a href="javascript:">移入收藏夹</a></p>
                  <p>
                    <a
                      class="green"
                      href="javascript:"
                      @click="deleteGoodsOfCartBySkuId(item.skuId)"
                      >删除</a
                    >
                  </p>
                  <p><a href="javascript:">找相似</a></p>
                </td>
              </tr>
            </tbody>
            <!-- 无效商品 -->
            <tbody>
              <tr>
                <td colspan="6"><h3 class="tit">失效商品</h3></td>
              </tr>
              <tr v-for="item in invalidGoodsList" :key="item">
                <td></td>
                <td>
                  <div class="goods">
                    <RouterLink :to="`/goods/${item.id}`"
                      ><img :src="item.picture" alt=""
                    /></RouterLink>
                    <div>
                      <p class="name ellipsis">
                        {{ item.name }}
                      </p>
                      <p class="attr">{{ item.attrsText }}</p>
                    </div>
                  </div>
                </td>
                <td class="tc">
                  <p>&yen;{{ item.nowPrice }}</p>
                </td>
                <td class="tc">{{ item.count }}</td>
                <td class="tc">
                  <p>
                    &yen;{{ item.count * Number(item.nowPrice).toFixed(2) }}
                  </p>
                </td>
                <td class="tc">
                  <p>
                    <a class="green" href="javascript:">删除</a>
                  </p>
                  <p><a href="javascript:">找相似</a></p>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <!-- 操作栏 -->
        <div class="action">
          <div class="batch">
            <XtxCheckbox>全选</XtxCheckbox>
            <a
              @click="deleteGoodsOfCart('selectedGoodsList')"
              href="javascript:"
              >删除商品</a
            >
            <a href="javascript:">移入收藏夹</a>
            <a @click="deleteGoodsOfCart('invalidGoodsList')" href="javascript:"
              >清空失效商品</a
            >
          </div>
          <div class="total">
            共 {{ effectiveGoodsCount }} 件商品，已选择
            {{ selectedGoodsCount }} 件，商品合计：
            <span class="red">¥{{ selectedGoodsPrice }}</span>
            <XtxButton type="primary" @click="jumpToCheckout"
              >下单结算</XtxButton
            >
          </div>
        </div>
        <!-- 猜你喜欢 -->
        <GoodsRelevant />
      </div>
    </div>
  </AppLayout>
</template>
<script>
import GoodsRelevant from "@/views/goods/components/GoodsRelevant";
import EmptyCart from "@/views/cart/components/EmptyCart";
import AppLayout from "@/components/AppLayout";
import { useStore } from "vuex";
import { computed } from "vue";
import Confirm from "@/components/library/confirm";
import Message from "@/components/library/message";
import CartSku from "@/views/cart/components/CartSku";
import { useRouter } from "vue-router";

export default {
  name: "CartPage",
  components: { CartSku, GoodsRelevant, AppLayout, EmptyCart },
  setup() {
    //获取store
    const store = useStore();
    //获取路由对象
    const router = useRouter();
    //更新商品购物车数据
    store.dispatch("cart/updateGoodsBySkuId");
    //有效商品列表
    const effectiveGoodsList = computed(
      () => store.getters["cart/effectiveGoodsList"]
    );
    //获取有效商品数量
    const effectiveGoodsCount = computed(
      () => store.getters["cart/effectiveGoodsCount"]
    );
    //获取无效商品列表
    const invalidGoodsList = computed(
      () => store.getters["cart/invalidGoodsList"]
    );
    //获取用户选择的商品总价
    const selectedGoodsPrice = computed(
      () => store.getters["cart/selectedGoodsPrice"]
    );
    //获取用户选择的商品数量
    const selectedGoodsCount = computed(
      () => store.getters["cart/selectedGoodsCount"]
    );
    //获取全选按钮状态
    const selectAllButtonStatus = computed(
      () => store.getters["cart/selectedAllButtonStatus"]
    );
    //删除购物车项
    const deleteGoodsOfCartBySkuId = (skuId) => {
      Confirm({
        content: "确认删除？",
      })
        .then(() => {
          store.dispatch("cart/deleteGoodsOfCart", skuId);
        })
        .catch(() => {});
    };
    //删除商品
    const deleteGoodsOfCart = (flag) => {
      let content = "";
      if (flag === "selectedGoodsList") {
        if (selectedGoodsCount.value === 0) {
          Message({ type: "warn", text: "至少选中一项商品" });
          return;
        }
        //批量删除用户选择商品
        content = "确定删除选中商品吗";
      } else if (flag === "invalidGoodsList") {
        if (invalidGoodsList.value.length === 0) {
          Message({ type: "warn", text: "没有无效商品" });
          return;
        }
        //删除无效商品
        content = "确定删除无效商品吗";
      }
      Confirm({ content }).then(() => {
        store.dispatch("cart/deleteManyGoodsOfCart", flag);
      });
    };
    //跳转到结算页面
    const jumpToCheckout = () => {
      //判断购物车是否存在已选择到商品
      if (selectedGoodsCount.value === 0) {
        Message({ type: "warn", text: "请选择商品" });
        return;
      }
      //跳转到结算页面
      router.push("/checkout/order");
    };
    return {
      effectiveGoodsList,
      effectiveGoodsCount,
      invalidGoodsList,
      selectedGoodsPrice,
      selectedGoodsCount,
      selectAllButtonStatus,
      deleteGoodsOfCartBySkuId,
      deleteGoodsOfCart,
      jumpToCheckout,
    };
  },
};
</script>
<style scoped lang="less">
.tc {
  text-align: center;
  .xtx-number-box {
    margin: 0 auto;
    width: 120px;
  }
}
.red {
  color: @priceColor;
}
.green {
  color: @xtxColor;
}
.f16 {
  font-size: 16px;
}
.goods {
  display: flex;
  align-items: center;
  img {
    width: 100px;
    height: 100px;
  }
  > div {
    width: 280px;
    font-size: 16px;
    padding-left: 10px;
    .attr {
      font-size: 14px;
      color: #999;
    }
  }
}
.action {
  display: flex;
  background: #fff;
  margin-top: 20px;
  height: 80px;
  align-items: center;
  font-size: 16px;
  justify-content: space-between;
  padding: 0 30px;
  .xtx-checkbox {
    color: #999;
  }
  .batch {
    a {
      margin-left: 20px;
    }
  }
  .red {
    font-size: 18px;
    margin-right: 20px;
    font-weight: bold;
  }
}
.tit {
  color: #666;
  font-size: 16px;
  font-weight: normal;
  line-height: 50px;
}
.cart-page {
  .cart {
    background: #fff;
    color: #666;
    table {
      border-spacing: 0;
      border-collapse: collapse;
      line-height: 24px;
      width: 100%;
      th,
      td {
        padding: 10px;
        border-bottom: 1px solid #f5f5f5;
        &:first-child {
          text-align: left;
          padding-left: 30px;
          color: #999;
        }
      }
      th {
        font-size: 16px;
        font-weight: normal;
        line-height: 50px;
      }
    }
  }
}
</style>
