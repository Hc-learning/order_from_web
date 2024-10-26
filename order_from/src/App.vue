<template>
  <div class="max-box">
    <dir>
      <h1>订单管理</h1>
      <el-tabs v-model="activeName" class="demo-tabs" @tab-click="handleClick">
        <el-tab-pane label="全部" name="first">
          <div style="padding: 20px;margin-right: 30px;background-color: white;">
            <div class="order-status" @click="onItem" style="display: flex;align-items: center;">
              <div style="margin-right: 10px;">订单状态</div>
              <el-button id="active" @click="onAll()" class="all">全部</el-button>
              <el-button @click="onAll('运输中')">运输中 {{ yun }}</el-button>
              <el-button @click="onAll('已送达')">已送达 {{ da }}</el-button>
            </div>
            <div class="items-box">
              <el-form :inline="true" :model="formInline" ref="ruleFormRef" class="demo-form-inline">
                <el-form-item label="订单ID" prop="id">
                  <el-input v-model="formInline.id" placeholder="请输入ID" clearable />
                </el-form-item>
                <el-form-item label="物流方式" prop="value">
                  <el-select v-model="formInline.value" placeholder="请选择物流方式" style="width: 240px">
                    <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
                  </el-select>
                </el-form-item>
                <el-form-item label="订单数量" prop="bq">
                  <el-input v-model="formInline.bq" placeholder="请输入订单数量" clearable />
                </el-form-item>
                <el-form-item label="价钱之内" prop="money">
                  <el-input v-model="formInline.money" placeholder="请输入价钱" clearable />
                </el-form-item>
                <el-form-item>
                  <el-button @click="resetForm(ruleFormRef)">重置</el-button>
                </el-form-item>
              </el-form>
            </div>
          </div>
          <div style="margin: 10px 30px 0 0;padding: 20px;background-color: white;">
            <el-row>
              <el-col :span="18">
                <div>
                  <span style="font-size: 10px;margin-right: 5px;">{{ total_ }} 订单</span>
                  <el-button @click="onExport()">导出订单</el-button>
                  <el-button @click="exportToJSON()">导出记录</el-button>
                </div>
              </el-col>
              <el-col :span="4">
                <div>
                  <el-button @click="onSort" :id="isSort ? 'active-date' : ''">创建时间由近到远</el-button>
                </div>
              </el-col>
            </el-row>
          </div>
          <div>
            <div style="font-size: 11px;margin: 20px 30px;">
              <el-row>
                <el-col :span="12"><span>订单ID</span></el-col>
                <el-col :span="4"><span>订单状态</span></el-col>
                <el-col :span="4"><span>物流</span></el-col>
                <el-col :span="4"><span>支付</span></el-col>
              </el-row>
            </div>
            <div style="background-color: white;padding: 10px;margin-right: 30px;">
              <template v-for="item in get_new_list_data()" :key="item.Platform">
                <div @click="onId(item.Platform)" :data-id="item.Platform" class="order-list"
                  style="margin-top: 10px;padding: 10px;cursor: pointer;">
                  <div style="padding-bottom: 20px;font-size: 14px; color:#B0B0B0 ;border-bottom: 1px solid #B0B0B0;">
                    ID:{{ item.Platform }}</div>
                  <el-row>
                    <el-col :span="12">
                      <div class="order-item">
                        <div>
                          <img src="../public/OIP-C.jpg" style="width: 40px;height: 60px;" alt="">
                        </div>
                        <div class="order-item-content">
                          <div class="order-item-title">
                            <span style="width: 230px;height: 20px;">{{ item.name }}</span>
                            <span style="margin-top: 5px;color: #B0B0B0;font-size: 13px;">{{ item.zp ? item.totle + ' '
                              +
                              item.zp : ''
                              }}</span>
                          </div>
                          <span style="margin-left: 5px;">x{{ item.totle }}</span>
                        </div>
                      </div>
                    </el-col>
                    <el-col :span="4">
                      <div style="margin-top: 10px;">{{ item.zt }}</div>
                    </el-col>
                    <el-col :span="4">
                      <div style="margin-top: 10px;">{{ item.wl }}</div>
                    </el-col>
                    <el-col :span="4">
                      <div style="margin-top: 10px;">
                        <div>总金额：${{ item.je }}</div>
                        <div style="font-size: 14px;color: #B0B0B0;">{{ item.date.toLocaleString() }}</div>
                      </div>
                    </el-col>
                  </el-row>
                </div>
              </template>
            </div>
            <div style="background-color: white;margin: 20px 30px 0 0;padding: 20px;text-align: center;"
              v-if="total_ === 0">没有查询到数据
            </div>
            <el-pagination style="display: flex;justify-content: center;margin-top: 20px;"
              :current-page="Page.currentPage" :page-size="Page.pageSize" :total="total_"
              @current-change="handleCurrentChange" size="small" background layout="prev, pager, next" class="mt-4" />
          </div>
        </el-tab-pane>
        <el-tab-pane id="detail" label="详细页" name="detail">
          <div class="order-detail">
            <img src="../public/OIP-C.jpg" alt="">
            <div>
              <div>商品名称：{{ detail_data.name }}</div>
              <div>赠品：{{ detail_data.zp ? detail_data.totle + ' ' + detail_data.zp : '' }}</div>
              <div>物流：{{ detail_data.wl }}</div>
              <div>订单状态：{{ detail_data.zt }}</div>
              <div>总金额：${{ detail_data.je }}</div>
              <div>创建时间：{{ detail_data.date.toLocaleString() }}</div>
            </div>
          </div>
        </el-tab-pane>
        <el-tab-pane label="代发货" name="third">
          <el-empty :image-size="400" description="No Data" />
        </el-tab-pane>
        <el-tab-pane label="已发货" name="fourth">
          <el-empty :image-size="400" description="No Data" />
        </el-tab-pane>
        <el-tab-pane label="已完成" name="wc">
          <el-empty :image-size="400" description="No Data" />
        </el-tab-pane>
        <el-tab-pane label="已取消" name="qx">
          <el-empty :image-size="400" description="No Data" />
        </el-tab-pane>
      </el-tabs>
    </dir>
  </div>
</template>
<script setup>
import { ref, reactive } from 'vue'
import * as XLSX from 'xlsx';
// 搜索框表格数据
const ruleFormRef = ref(); // 表单引用

// 初始化页面选择栏
const activeName = ref('first'); // 当前活动标签

// 分页数据
const Page = ref({
  currentPage: 1, // 当前页码
  pageSize: 5, // 每页显示数量
})

// 分页事件
const handleCurrentChange = (newPage) => Page.value.currentPage = newPage; // 处理页码变化

// 按钮选中状态
function onItem(event) {
  if (!event) return; // 如果没有事件，直接返回
  if (event.target.tagName === 'BUTTON') {
    clear_id(event, 'BUTTON') // 按钮点击时清除ID
  } else if (event.target.tagName === 'SPAN') {
    clear_id(event, 'SPAN') // 其他元素点击时清除ID
  }

}

// 按钮状态事件
function clear_id(element, str = '') {
  if (!element) return // 如果没有元素，直接返回
  document.getElementById('active').id = ''; // 清除当前激活按钮的ID
  if (str) {
    if (str === 'BUTTON') {
      element.target.id = 'active'; // 设置按钮的ID为活跃状态
    } else if (str === 'SPAN') {
      element.target.parentNode.id = 'active'; // 设置SPAN父元素的ID为活跃状态
    }
  } else {
    element.id = 'active'; // 设置元素的ID为活跃状态
  }
}

// 表单数据
const formInline = reactive({
  id: '', // 订单ID
  value: '', // 物流方式
  bq: '', // 订单数量
  money: '' // 价格
})

// 订单数据
const list_data = [
  { Platform: '576751890783047968', zp: 'bottle', name: 'ENVIPRO Sea Moss Advanced Blend, 19789mg with Ashwagandha, Black Seed Oil & Ginger, Multivitamin Sleep Supplement , Immune Boost & Energy support health', totle: '1', zt: '运输中', wl: '商家物流', je: '21.88', date: new Date('2024-07-05 10:00:00') },
  { Platform: '576750433616302282', name: 'ENVIPRO Wellness Boost Supplement: Natural Dietary Powder for Fitness, Health Support & Daily Vitality ashwagandhaextract', totle: '1', zt: '运输中', wl: '商家物流', je: '26.88', date: new Date('2024-07-06 10:00:00') },
  { Platform: '576750433567740767', zp: 'BAGS', name: 'ENVIPRO Dual Sleep Support Combo: Wellness Boost & Sea Moss Advanced Blend multivitamins ashwagandhaextract', totle: '4', zt: '已送达', wl: '买家物流', je: '300.88', date: new Date('2024-07-06 11:00:00') },
  { Platform: '576749917006697381', name: 'ENVIPRO Dual Sleep Support Combo: Wellness Boost & Sea Moss Advanced Blend multivitamins ashwagandhaextract', totle: '1', zt: '已送达', wl: '商家物流', je: '21.88', date: new Date('2024-08-15 10:00:00') },
  { Platform: '576749914523472740', zp: 'SEAMOSS', name: 'ENVIPRO Wellness Boost Supplement: Natural Dietary Powder for Fitness, Health Support & Daily Vitality ashwagandhaextract', totle: '3', zt: '运输中', wl: '商家物流', je: '210.88', date: new Date('2024-08-09 10:00:00') },
  { Platform: '576749901241225658', zp: 'BAG', name: 'ENVIPRO Wellness Boost Supplement: Natural Dietary Powder for Fitness, Health Support & Daily Vitality ashwagandhaextract', totle: '2', zt: '已送达', wl: '买家物流', je: '188.88', date: new Date('2024-08-11 10:00:00') },
  { Platform: '576750433010815818', name: 'ENVIPRO Wellness Boost Supplement: Natural Dietary Powder for Fitness, Health Support & Daily Vitality ashwagandhaextract', totle: '2', zt: '运输中', wl: '商家物流', je: '99.88', date: new Date('2024-08-10 10:00:00') },
  { Platform: '576751890783047969', zp: 'bottle', name: 'ENVIPRO Omega-3 Fish Oil, 1000mg, Heart Health & Brain Support', totle: '2', zt: '运输中', wl: '商家物流', je: '15.00', date: new Date('2024-09-01 10:00:00') },
  { Platform: '576751890783047970', name: 'ENVIPRO Turmeric Curcumin, 1000mg, Joint Support Supplement', totle: '1', zt: '已送达', wl: '买家物流', je: '18.50', date: new Date('2024-09-02 10:00:00') },
  { Platform: '576751890783047971', zp: 'BOTTLE', name: 'ENVIPRO Ashwagandha Root Extract, Stress Relief & Mood Support', totle: '3', zt: '运输中', wl: '商家物流', je: '24.99', date: new Date('2024-09-03 11:00:00') },
  { Platform: '576751890783047972', name: 'ENVIPRO Superfood Greens Powder, Nutrient-Rich Supplement', totle: '1', zt: '已送达', wl: '买家物流', je: '29.99', date: new Date('2024-09-04 12:00:00') },
  { Platform: '576751890783047973', zp: 'BAGS', name: 'ENVIPRO Keto Meal Replacement Shake, Vanilla Flavor', totle: '4', zt: '运输中', wl: '商家物流', je: '50.00', date: new Date('2024-09-05 10:00:00') },
  { Platform: '576751890783047974', name: 'ENVIPRO Sleep Support, Melatonin & Natural Herbs', totle: '3', zt: '已送达', wl: '买家物流', je: '20.75', date: new Date('2024-09-06 10:00:00') },
  { Platform: '576751890783047975', zp: 'VITAMINS', name: 'ENVIPRO Daily Multivitamin for Men, Comprehensive Formula', totle: '2', zt: '运输中', wl: '商家物流', je: '22.50', date: new Date('2024-09-07 09:00:00') },
  { Platform: '576751890783047976', name: 'ENVIPRO Women’s Multivitamin with Probiotics', totle: '2', zt: '已送达', wl: '买家物流', je: '27.00', date: new Date('2024-09-08 10:00:00') },
  { Platform: '576751890783047977', zp: 'BOTTLE', name: 'ENVIPRO Digestive Enzymes for Gut Health Support', totle: '1', zt: '运输中', wl: '商家物流', je: '23.00', date: new Date('2024-09-09 11:00:00') },
  { Platform: '576751890783047978', name: 'ENVIPRO Probiotic Gummies for Digestive Health', totle: '2', zt: '已送达', wl: '买家物流', je: '30.00', date: new Date('2024-09-10 12:00:00') },
  { Platform: '576751890783047979', zp: 'PACK', name: 'ENVIPRO Vitamin D3 & K2, Immune System Booster', totle: '1', zt: '运输中', wl: '商家物流', je: '21.20', date: new Date('2024-09-11 10:00:00') },
  { Platform: '576751890783047980', name: 'ENVIPRO Protein Powder, Chocolate Flavor, Muscle Recovery', totle: '3', zt: '已送达', wl: '买家物流', je: '45.00', date: new Date('2024-09-12 10:00:00') },
  { Platform: '576751890783047981', zp: 'BAGS', name: 'ENVIPRO Meal Prep Containers, BPA-Free, 3 Compartment', totle: '5', zt: '运输中', wl: '商家物流', je: '35.00', date: new Date('2024-09-13 11:00:00') },
  { Platform: '576751890783047982', name: 'ENVIPRO Fit a Mastectomy Bra, Post-Surgery Comfort', totle: '1', zt: '已送达', wl: '买家物流', je: '55.00', date: new Date('2024-09-14 12:00:00') },
  { Platform: '576751890783047983', zp: 'BOTTLE', name: 'ENVIPRO Plant Protein Powder, Vanilla Flavor', totle: '2', zt: '运输中', wl: '商家物流', je: '40.50', date: new Date('2024-09-15 10:00:00') },
  { Platform: '576751890783047984', name: 'ENVIPRO Nut Butter Pack, Almond & Cashew Mix', totle: '3', zt: '已送达', wl: '买家物流', je: '22.30', date: new Date('2024-09-16 10:00:00') },
  { Platform: '576751890783047985', zp: 'BAGS', name: 'ENVIPRO Fitness Bands Set for Home Workouts', totle: '1', zt: '运输中', wl: '商家物流', je: '18.75', date: new Date('2024-09-17 09:00:00') },
  { Platform: '576751890783047986', name: 'ENVIPRO Yoga Mat, Non-Slip, Eco-Friendly', totle: '2', zt: '已送达', wl: '买家物流', je: '30.00', date: new Date('2024-09-18 10:00:00') },
  { Platform: '576751890783047987', zp: 'VITAMINS', name: 'ENVIPRO Collagen Peptides for Healthy Skin', totle: '1', zt: '运输中', wl: '商家物流', je: '29.99', date: new Date('2024-09-19 12:00:00') },
  { Platform: '576751890783047988', name: 'ENVIPRO Apple Cider Vinegar, Gummies, Weight Management', totle: '2', zt: '已送达', wl: '买家物流', je: '23.30', date: new Date('2024-09-20 10:00:00') },
  { Platform: '576751890783047989', zp: 'PACK', name: 'ENVIPRO Organic Chia Seeds, 16oz', totle: '3', zt: '运输中', wl: '商家物流', je: '15.45', date: new Date('2024-09-21 11:00:00') },
  { Platform: '576751890783047990', name: 'ENVIPRO Organic Green Tea, Loose Leaf', totle: '1', zt: '已送达', wl: '买家物流', je: '10.50', date: new Date('2024-09-22 10:00:00') },
  { Platform: '576751890783047991', zp: 'BOTTLE', name: 'ENVIPRO Elderberry Gummies, Immune Support', totle: '2', zt: '运输中', wl: '商家物流', je: '12.00', date: new Date('2024-09-23 09:00:00') },
  { Platform: '576751890783047992', name: 'ENVIPRO Fiber Supplement, Natural, for Digestive Health', totle: '3', zt: '已送达', wl: '买家物流', je: '24.99', date: new Date('2024-09-24 10:00:00') },
  { Platform: '576751890783047993', zp: 'BAGS', name: 'ENVIPRO Healthy Snack Pack, Trail Mix, 3-Pack', totle: '4', zt: '运输中', wl: '商家物流', je: '15.00', date: new Date('2024-09-25 10:00:00') },
  { Platform: '576751890783047994', name: 'ENVIPRO Ginseng Root Extract, Energy Booster', totle: '1', zt: '已送达', wl: '买家物流', je: '29.99', date: new Date('2024-09-26 11:00:00') },
  { Platform: '576751890783047995', zp: 'PACK', name: 'ENVIPRO Organic Coconut Oil, 16oz', totle: '5', zt: '运输中', wl: '商家物流', je: '22.50', date: new Date('2024-09-27 10:00:00') },
  { Platform: '576751890783047996', name: 'ENVIPRO Vegan Protein Bars, Peanut Butter Flavor', totle: '2', zt: '已送达', wl: '买家物流', je: '18.00', date: new Date('2024-09-28 10:00:00') },
  { Platform: '576751890783047997', zp: 'BOTTLE', name: 'ENVIPRO Anti-Stress Herbal Tea, Calming Effect', totle: '1', zt: '运输中', wl: '商家物流', je: '10.99', date: new Date('2024-09-29 09:00:00') },
  { Platform: '576751890783047998', name: 'ENVIPRO Essential Oil Set for Aromatherapy', totle: '3', zt: '已送达', wl: '买家物流', je: '34.99', date: new Date('2024-09-30 10:00:00') },
  { Platform: '576751890783047999', zp: 'VITAMINS', name: 'ENVIPRO Magnesium Citrate, Muscle Relaxation', totle: '2', zt: '运输中', wl: '商家物流', je: '15.45', date: new Date('2024-10-01 10:00:00') },
  { Platform: '576751890783048000', zp: 'BOTTLE', name: 'ENVIPRO Collagen Gummies, Hair, Skin & Nails Support', totle: '2', zt: '运输中', wl: '商家物流', je: '28.00', date: new Date('2024-10-02 10:00:00') },
  { Platform: '576751890783048001', name: 'ENVIPRO Organic Protein Powder, Vanilla Bean', totle: '1', zt: '已送达', wl: '买家物流', je: '40.00', date: new Date('2024-10-03 10:00:00') },
  { Platform: '576751890783048002', zp: 'PACK', name: 'ENVIPRO Chia Seed Pudding mix, Chocolate Flavor', totle: '3', zt: '运输中', wl: '商家物流', je: '19.99', date: new Date('2024-10-04 11:00:00') },
  { Platform: '576751890783048003', name: 'ENVIPRO Keto-Friendly Snack Box, Variety Pack', totle: '4', zt: '已送达', wl: '买家物流', je: '35.50', date: new Date('2024-10-05 12:00:00') },
  { Platform: '576751890783048004', zp: 'BAGS', name: 'ENVIPRO Gluten-Free Brownie Mix, Healthy Treat', totle: '1', zt: '运输中', wl: '商家物流', je: '12.00', date: new Date('2024-10-06 10:00:00') },
  { Platform: '576751890783048005', name: 'ENVIPRO Vegan Chocolate Protein Bars, Snack Pack', totle: '3', zt: '已送达', wl: '买家物流', je: '32.00', date: new Date('2024-10-07 10:00:00') },
  { Platform: '576751890783048006', zp: 'BOTTLE', name: 'ENVIPRO Coconut Water, Organic, Hydration', totle: '2', zt: '运输中', wl: '商家物流', je: '3.99', date: new Date('2024-10-08 11:00:00') },
  { Platform: '576751890783048007', name: 'ENVIPRO Superfood Smoothie Mix, Berry Flavor', totle: '1', zt: '已送达', wl: '买家物流', je: '29.00', date: new Date('2024-10-09 10:00:00') },
  { Platform: '576751890783048008', zp: 'PACK', name: 'ENVIPRO Plant-Based Snacks, Variety Pack', totle: '5', zt: '运输中', wl: '商家物流', je: '40.00', date: new Date('2024-10-10 10:00:00') },
  { Platform: '576751890783048009', name: 'ENVIPRO Detox Tea, Herbal Blend, Cleanse Support', totle: '2', zt: '已送达', wl: '买家物流', je: '22.50', date: new Date('2024-10-11 10:00:00') },
  { Platform: '576751890783048010', zp: 'BAGS', name: 'ENVIPRO Protein Bites, Peanut Butter Chocolate', totle: '3', zt: '运输中', wl: '商家物流', je: '15.75', date: new Date('2024-10-12 09:00:00') },
  { Platform: '576751890783048011', name: 'ENVIPRO Healthy Omega-3 Snack Bars, Maple Flavor', totle: '1', zt: '已送达', wl: '买家物流', je: '18.00', date: new Date('2024-10-13 10:00:00') }
];

// 详细页页面初始化数据
let detail_data = ref(list_data[0]); // 详情数据的引用
// 点击订单列表获得详细数据
function onId(_id) {
  detail_data.value = list_data.find(item => item.Platform === _id); // 找到对应的订单数据
  activeName.value = "detail" // 切换到详细页
}

// 总数
let total_ = 1; // 当前订单总数
// 运输状态
let x = ref(); // 当前运输状态
// 是否排序
let isSort = ref(false); // 排序状态
// 导出数据
let push_data = []; // 导出的数据
// 初始化页面数据
function get_new_list_data() {
  // 将原数组复制一份
  let new_list_data = [...list_data]; // 复制原始订单数据
  // 判断不成功返回原数组
  // x -> 运输状态
  // formInline -> 表单数据
  if (x.value && !formInline.id && !formInline.value && !formInline.bq && !formInline.money) {
    new_list_data = list_data.filter(item => item.zt === x.value) // 过滤运输状态
  } else {
    // 筛选数据
    // 将x.value赋值为空
    x.value = '';
    // 订单ID筛选
    if (formInline.id) new_list_data = new_list_data.filter(({ Platform }) => Platform === formInline.id); // 筛选订单ID
    // 物流方式筛选
    if (formInline.value) new_list_data = new_list_data.filter(({ wl }) => formInline.value === wl); // 筛选物流方式
    // 订单数量筛选
    if (formInline.bq) new_list_data = new_list_data.filter(({ totle }) => formInline.bq === totle); // 筛选订单数量
    // 金额筛选
    if (formInline.money) new_list_data = new_list_data.filter(({ je }) => parseFloat(formInline.money) >= parseFloat(je)).sort((a, b) => parseFloat(a.je) - parseFloat(b.je)); // 筛选金额
    // 将全部按钮设置为选择状态
    clear_id(document.querySelector('.all')) // 清空全部按钮的ID
  }
  // 计算总数
  total_ = new_list_data.length; // 获取筛选后的总数据量
  // 导出的数据
  push_data = new_list_data; // 更新导出数据
  // 按时间排序
  new_list_data = isSort.value ? new_list_data.sort((a, b) => b.date - a.date) : new_list_data; // 根据日期排序
  // 页面分页 只有超过一页才分页
  if (total_ > 6) {
    // 页面分页
    new_list_data = new_list_data.slice((Page.value.currentPage - 1) * Page.value.pageSize, Page.value.currentPage * Page.value.pageSize); // 进行分页处理
  }
  return new_list_data // 返回处理后的数据
}

// 页面初始化
get_new_list_data(); // 初始化数据

// 物流方式
const options = [
  { value: '商家物流', label: '商家物流' }, // 物流选项
  { value: '买家物流', label: '买家物流' } // 物流选项
];

// 重置表单
function resetForm(formEl) {
  if (!formEl) return; // 如果没有表单元素，直接返回
  formEl.resetFields() // 重置表单字段
}

// 运输中数量
let yun = 0; // 运输中的订单数量
// 已送达数量
let da = 0; // 已送达的订单数量
// 写入数量
for (let i = 0; i < list_data.length; i++) {
  if (list_data[i].zt === '运输中') {
    yun++; // 统计运输中订单数量
  }
  if (list_data[i].zt === '已送达') {
    da++; // 统计已送达订单数量
  }
}

// 按钮事件
function onAll(val=false) {
  x.value = val; // 按照输入的内容进行筛选
  Page.value.currentPage = 1; // 重置当前页数为1
}

// 时间排序按钮事件
function onSort() {
  isSort.value = !isSort.value; // 切换排序状态
}

// 导出订单按钮事件
function onExport() {
  const ws = XLSX.utils.json_to_sheet(push_data); // 将数据转换为工作表
  const wb = XLSX.utils.book_new(); // 创建新的工作簿
  XLSX.utils.book_append_sheet(wb, ws, 'Sheet1'); // 将工作表加入工作簿
  XLSX.writeFile(wb, '订单列表.xlsx'); // 导出Excel文件
}

// 导出记录按钮事件
function exportToJSON() {
  const jsonData = JSON.stringify(push_data, null, 2); // 转换为格式化的JSON字符串

  // 创建JSON文件内容
  const blob = new Blob([jsonData], { type: 'application/json;charset=utf-8;' }); // 创建Blob文件
  const url = URL.createObjectURL(blob); // 创建下载链接

  // 创建一个临时链接元素
  const a = document.createElement('a');
  a.setAttribute('href', url);
  a.setAttribute('download', '订单数据.txt'); // 下载文件名
  a.style.visibility = 'hidden';
  document.body.appendChild(a);
  a.click(); // 触发点击事件下载文件
  document.body.removeChild(a); // 清理临时元素
}
</script>

<style>
.order-detail {
  height: 300px;
  display: flex;
}

.order-detail img {
  width: 300px;
  height: 200px;
  margin-right: 20px;
  display: flex;
  align-items: center;
  align-self: center;
  margin-left: 10px;
}

.order-detail div {
  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
  justify-content: space-around;
}

#active {
  background-color: #cdddf5;
  border: 1px solid #7baaf1;
  color: #4488f0;
}

#active-date {
  background-color: #cdddf5;
  border: 1px solid #7baaf1;
  color: #4488f0;
}

.order-list:hover {
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  border-radius: 2%;
}


.el-button:focus {
  outline: none;
}

.order-item {
  display: flex;
  flex-direction: row;
  margin-top: 10px;
}

.order-item-title {
  display: flex;
  flex-direction: column;
  margin: 0 10px 0 10px;
}

.order-item-content {
  display: flex;
  justify-content: center;
  align-items: start;

}


.max-box {
  min-width: 860px;
  position: absolute;
  width: 100%;
  background-color: #ebebeb;

  .items-box {
    background-color: rgb(255, 255, 255);
    display: flex;
    justify-content: center;
    align-items: center;
    padding-top: 20px;
  }
}

span {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;

}

.base-table {
  border-radius: 5px;
  background: #ffffff;
  margin-top: 20px;
  margin-bottom: 20px;
}

.action {
  border-radius: 5px 5px 0px 0px;
  background: #ffffff;
  padding: 20px;
  border-bottom: 1px solid #ece8e8;
}

.pagination {
  justify-content: flex-end;
  padding: 14px;
}
</style>