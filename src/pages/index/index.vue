<script setup lang="ts">
import { onLoad } from '@dcloudio/uni-app'
import type { BannerItem, CategoryItem, HotItem } from '@/types/home'
import CustormNavbar from './components/CustormNavbar.vue'
import CategoryPanel from './components/CategoryPanel.vue'
import HotPanel from './components/HotPanel.vue'
import PageSkeleton from './components/PageSkeleton.vue'
import { getHomeBannerAPI } from '@/services/home.ts'
import { ref } from 'vue'
import { getHomeCategoryAPI, getHomeHotAPI } from '@/services/home'
import { useGuessList } from '@/composables/index.ts'
//获取轮播图数据
const bannerList = ref<BannerItem[]>([])
const getHomeBannerData = async () => {
  const res = await getHomeBannerAPI()
  console.log(res)
  bannerList.value = res.result
}
//获取前台分类数据
const categoryList = ref<CategoryItem[]>()
const getHomeCategory = async () => {
  const res = await getHomeCategoryAPI()
  categoryList.value = res.result
}
//获取热门推荐数据
const homeHotList = ref<HotItem[]>()
const getHomeHotData = async () => {
  const res = await getHomeHotAPI()
  homeHotList.value = res.result
}
// 是否加载中的标记
const isLoading = ref(false)
// 页面加载
onLoad(async () => {
  isLoading.value = true
  await Promise.all([getHomeBannerData(), getHomeCategory(), getHomeHotData()])
  isLoading.value = false
})
const { guessRef, onScrolltolower } = useGuessList()

const isTriggered = ref(false)
// 自定义导航栏
const onRefresherrefresh = async () => {
  // 开始动画
  isTriggered.value = true
  // 加载数据
  // 这样写是请求一个接着一个发送，等待时间会变长
  // await getHomeBannerData()
  // await getHomeBannerData()
  // await getHomeHotData()
  //重置猜你喜欢数据
  guessRef.value?.resetData()
  // 同时发送请求用Promise.all
  await Promise.all([
    getHomeBannerData(),
    getHomeBannerData(),
    getHomeHotData(),
    guessRef.value?.getMore(),
  ])
  // 关闭动画
  isTriggered.value = false
}
</script>

<template>
  <!-- 自定义导航栏 -->
  <CustormNavbar />
  <scroll-view
    refresher-enabled="true"
    @refresherrefresh="onRefresherrefresh"
    :refresher-triggered="isTriggered"
    @scrolltolower="onScrolltolower"
    class="scroll-view"
    scroll-y
  >
    <PageSkeleton v-if="isLoading" />
    <template v-else>
      <!-- 自定义轮播图 -->
      <XtxSwiper :list="bannerList" />
      <!-- 分类面板 -->
      <CategoryPanel :list="categoryList" />
      <!-- 热门推荐 -->
      <HotPanel :list="homeHotList" />
      <!-- 猜你喜欢组件 -->
      <XtxGuess ref="guessRef" />
    </template>
  </scroll-view>
</template>

<style lang="scss">
page {
  background-color: #f7f7f7;
  height: 100%;
  display: flex;
  flex-direction: column;
}
.scroll-view {
  flex: 1;
}
</style>
