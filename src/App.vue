<template>
  <div class="layout">
    <div class="head">
      <div class="tittle">
        &nbsp;
      </div>
      <div class="switch">
        <div @click="resetPage(true)" class="in bei">&nbsp&nbsp</div>
        <div @click="resetPage(false)" class="in nan">&nbsp&nbsp</div>
        <div @click="togglePlay" class="in speaker">&nbsp</div> <!-- 喇叭按钮 -->
        <audio ref="audio" src=".\public\bgm\bgm.mp3" loop></audio> <!-- 音乐文件 -->
      </div>
    </div>
    <div class="container">
      <div class="map">
        <div class="svg-container" @mousedown="startDrag" @mouseup="endDrag" @mousemove="drag">
            <svg class="svg-content" :viewBox="computedViewBox" @mousemove="updateTooltipPosition" @mouseleave="hideTooltip" ref="mapSvg">
            <defs>
              <radialGradient id="highlightGradient" cx="50%" cy="50%" r="50%" fx="50%" fy="50%">
                <stop offset="0%" style="stop-color: #994925; stop-opacity:1" />
                <stop offset="100%" style="stop-color: #994925; stop-opacity:0" />
              </radialGradient>
            </defs>
            <!-- 切换地图 -->
            <image :href="computedHref" />
            <!-- 切换地图坐标 -->
            <circle v-for="circle in computedCircles"
                    :key="circle.id"
                    :cx="circle.cx"
                    :cy="circle.cy"
                    :r="circle.r / 1.5"
                    :class="{ highlighted: circle.highlighted }"
                    @mouseover="showTooltip(circle)"
                    @mouseleave="hideTooltip" >
              <title>{{ circle.title }}</title>
            </circle>
          </svg>
          <Tooltip v-if="tooltip.visible" :x="tooltip.x" :y="tooltip.y" :info="tooltip.info" />
        </div>
      </div>
      <div class="left">
        <div class="selectedframe">
          <!-- 庙号 -->
          <div class="lefttittle lefticon1">&nbsp;</div>
          <div class="box up">
            <div v-if="showBei" v-for="(item, index) in bei" :key="'bei' + index" class="list-item">
              <input type="checkbox" :id="'bei' + index" :value="item.name" v-model="selectedEmperors" @change="updateNianhao(item)" />
              <label :for="'bei' + index">{{ item.name.split(" ")[0] }}</label>
            </div>
            <div v-if="!showBei" v-for="(item, index) in nan" :key="'nan' + index" class="list-item">
              <input type="checkbox" :id="'nan' + index" :value="item.name" v-model="selectedEmperors" @change="updateNianhao(item)" />
              <label :for="'nan' + index">{{ item.name.split(" ")[0] }}</label>
            </div>
          </div>
          <!-- 年号 -->
          <div class="lefttittle lefticon2">&nbsp;</div>
          <div class="box down">
            <div v-if="showBei" v-for="(item, index) in displayedNianhao" :key="'nianhao' + index" class="list-item">
              <input type="checkbox" :id="'nianhao' + index" :value="item.label" v-model="selectedYears" @change="updateYears(circlesbei, item)"/>
              <label :for="'nianhao' + index">{{ item.label }}</label>
            </div>
            <div v-if="!showBei" v-for="(item, index) in displayedNianhao" :key="'nianhao' + index" class="list-item">
              <input type="checkbox" :id="'nianhao' + index" :value="item.label" v-model="selectedYears" @change="updateYears(circlesnan, item)"/>
              <label :for="'nianhao' + index">{{ item.label }}</label>
            </div>
          </div>
        </div>
        <div class="chatoutlook">
          <div class="chart">
            <BarChart :bars="bars" :width="500" :margin="20" :height="1400"/>
            <!-- <StackedBarChart :data="chartData" /> -->
          </div>
          <div class="legend">
            <div v-for="option in options" :key="option.id" class="legend-item">
              <span class="legend-color" :style="{ backgroundColor: option.value }"></span>
              <span class="legend-label">{{ option.label }}</span>
            </div>
          </div>
        </div>
      </div>
      <div :class="computedClass">&nbsp;</div>
    </div>
  </div>
</template>

<script>
import Tooltip from './components/Tooltip.vue';
import BarChart from './components/Barchart.vue';
import { image } from 'd3';
import "./assets/fonts/song.css";

export default {
  name: 'App',
  components: {
    // StackedBarChart,
    Tooltip,
    BarChart,
    image
  },
  data() {
    return {
      chartData: null,
      data: null,
      showBei: true,
      selectedOption: null,
      selectedItems: [],
      // 朝号年号数据
      selectedEmperors: [],
      selectedYears: [],
      displayedNianhao: [], 
      // 类别数据
      options: [
        { id: 'class1', value: '#C9C990', label: '经' },
        { id: 'class2', value: '#E4C58E', label: '史' },
        { id: 'class3', value: '#994925', label: '子' },
        { id: 'class4', value: '#F4E8D1', label: '集' },
        { id: 'class5', value: '#B6BEA5', label: '从' },
      ],
      // 地图坐标
      circlesbei: [
        {"id": 0.0, "cx": 1526.5, "cy": 812.5, "r": 15.0, "fill": "#AF563B", "attributes": ["宋仁宗 赵祯", "天圣"], "info": "归化镇彰教院\n河北东路（路）\n河间府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 1.0, "cx": 1520.5, "cy": 1312.5, "r": 15.0, "fill": "#AF563B", "attributes": [, "嘉定"], "info": "白鹭洲书院\n江南西路（路）\n吉州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 2.0, "cx": 1663.5, "cy": 1400.5, "r": 15.0, "fill": "#AF563B", "attributes": ["宋神宗 赵顼", "元丰"], "info": "新罗院\n福建路（路）\n漳州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 4.0, "cx": 1710.0, "cy": 1149.0, "r": 20.0, "fill": "#AF563B", "attributes": [, "嘉祐"], "info": "杭州钱家\n两浙路（路）\n杭州（州）\n\n刻书数目：2", "highlighted": false},
        {"id": 5.0, "cx": 1663.0, "cy": 1325.5, "r": 45.0, "fill": "#AF563B", "attributes": ["宋神宗 赵顼", "宋哲宗 赵煦", "宋徽宗 赵佶", "崇宁","大观", "建中靖国", "绍圣", "元丰", "元符", "元祐", "政和"], "info": "福州东禅等觉院\n福建路（路）\n福州（州）\n\n刻书数目：1345", "highlighted": false},
        {"id": 6.0, "cx": 1722.5, "cy": 1336.5, "r": 14.0, "fill": "#AF563B", "attributes": [], "info": "福州学官刻本\\n福建路（路）\n福州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 7.0, "cx": 1703.0, "cy": 1295.0, "r": 30.0, "fill": "#AF563B", "attributes": ["宋徽宗 赵佶", "宋钦宗 赵桓", "靖康", "宣和", "政和", "重和"], "info": "福州开元寺\n福建路（路）\n福州（州）\n\n刻书数目：253", "highlighted": false},
        {"id": 8.0, "cx": 1692.0, "cy": 1143.0, "r": 20.0, "fill": "#AF563B", "attributes": ["宋仁宗 赵祯", "宋神宗 赵顼", "熙宁", "庆历"], "info": "杭州晏家\n两浙路（路）\n杭州（州）\n\n刻书数目：2", "highlighted": false},
        {"id": 9.0, "cx": 1690.5, "cy": 1197.5, "r": 15.0, "fill": "#AF563B", "attributes": ["宋神宗 赵顼", "熙宁"], "info": "海盐县\n两浙路（路）\n秀州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 10.0, "cx": 1676.5, "cy": 1170.5, "r": 15.0, "fill": "#AF563B", "attributes": ["宋神宗 赵顼", "熙宁"], "info": "金栗山广惠禅院\n两浙路（路）\n嘉兴府（州）\n\n刻书数目：1", "highlighted": false},
      ],
      circlesnan: [
        {"id": 0, "cx": 1392.91, "cy": 1535.72, "r": 10, "attributes": [], "info": "眉阳\n成都府路（路）\n眉州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 1, "cx": 1404.66, "cy": 1498.1, "r": 10, "attributes": [], "info": "眉山程舍人\n成都府路（路）\n眉州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 2, "cx": 1751.36, "cy": 1516.91, "r": 10, "attributes": ["宋理宗 赵顼", "宝祐"], "info": "江陵府先锋隘李安桧\n荆湖北路（路）\n江宁府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 3, "cx": 1770.24, "cy": 1736.73, "r": 10, "attributes": ["宋孝宗 赵眘", "干道"], "info": "零陵郡\n荆湖南路（路）\n永州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 4, "cx": 1797.28, "cy": 1540.42, "r": 10, "attributes": ["宋高宗 赵构", "绍兴"], "info": "荆湖北路安抚使司\n荆湖北路（路）\n\n\n刻书数目：1", "highlighted": false},
        {"id": 5, "cx": 1849.0, "cy": 1512.21, "r": 14, "attributes": ["宋高宗 赵构","宋光宗 赵惇","宋宁宗 赵扩", "绍兴","绍熙","庆元"], "info": "黄州\n荆湖北路（路）\n鄂州（州）\n\n刻书数目：3", "highlighted": false},
        {"id": 6, "cx": 1876.03, "cy": 1522.79, "r": 10, "attributes": ["宋宁宗 赵扩", "庆元"], "info": "罗田县庠\n荆湖北路（路）\n鄂州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 7, "cx": 1878.38, "cy": 1489.88, "r": 10, "attributes": ["宋度宗 赵禥", "咸淳"], "info": "崇阳县斋\n荆湖北路（路）\n鄂州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 8, "cx": 1890.14, "cy": 1652.1, "r": 20, "attributes": [NaN, NaN], "info": "庐陵\n江南西路（路）\n吉州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 9, "cx": 1899.54, "cy": 1569.81, "r": 12, "attributes": ["宋孝宗 赵眘","宋宁宗 赵扩", "淳熙","嘉定"], "info": "江西转运司\n江南西路（路）\n\n刻书数目：2", "highlighted": false},
        {"id": 10, "cx": 1906.6, "cy": 1612.13, "r": 12, "attributes": ["宋孝宗 赵眘","宋宁宗 赵扩", "干道","庆元"], "info": "江西\n江南西路（路）\n\n刻书数目：2", "highlighted": false},
        {"id": 11, "cx": 1908.95, "cy": 1741.43, "r": 10, "attributes": ["宋高宗 赵构", "绍兴"], "info": "赣州\n江南西路（路）\n赣州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 12, "cx": 1911.3, "cy": 1703.81, "r": 12, "attributes": ["宋宁宗 赵扩", "嘉定"], "info": "章贡郡\n江南西路（路）\n赣州（州）\n\n刻书数目：2", "highlighted": false},
        {"id": 13, "cx": 1921.88, "cy": 1669.72, "r": 16, "attributes": ["宋孝宗 赵眘", "淳熙"], "info": "抚州公使库\n江南西路（路）\n抚州（州）\n\n刻书数目：4", "highlighted": false},
        {"id": 14, "cx": 1931.46, "cy": 1455.95, "r": 10, "attributes": ["宋孝宗 赵眘", "淳熙"], "info": "舒州公使库\n淮南西路（路）\n安庆军（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 15, "cx": 1934.81, "cy": 1565.11, "r": 10, "attributes": ["宋理宗 赵顼", "景定"], "info": "环溪书院刻仁斋\n江南西路（路）\n抚州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 16, "cx": 1941.47, "cy": 1491.01, "r": 10, "attributes": ["宋高宗 赵构", "绍兴"], "info": "无为军\n淮南西路（路）\n无为军（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 17, "cx": 1944.21, "cy": 1630.93, "r": 16, "attributes": ["宋光宗 赵惇", "淳熙"], "info": "抚州\n江南西路（路）\n抚州（州）\n\n刻书数目：4", "highlighted": false},
        {"id": 18, "cx": 1946.56, "cy": 1741.43, "r": 10, "attributes": ["宋光宗 赵惇", "淳熙"], "info": "南康郡\n江南西路（路）\n赣州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 19, "cx": 1949.52, "cy": 1703.72, "r": 10, "attributes": ["宋宁宗 赵扩", "嘉定"], "info": "兴国军学\n江南西路（路）\n赣州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 20, "cx": 1967.76, "cy": 1469.72, "r": 10, "attributes": ["宋宁宗 赵扩", "嘉定"], "info": "同安郡\n淮南西路（路）\n安庆军（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 21, "cx": 1972.42, "cy": 1586.27, "r": 10, "attributes": ["宋理宗 赵顼", "淳祐"], "info": "大庾县\n江南西路（路）\n南安军（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 22, "cx": 1977.13, "cy": 1663.85, "r": 10, "attributes": ["宋理宗 赵顼", "绍定"], "info": "临江军学\n江南西路（路）\n临江军（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 23, "cx": 1986.53, "cy": 1623.88, "r": 10, "attributes": ["宋孝宗 赵眘", "淳熙"], "info": "江西漕台\n江南西路（路）\n\n\n刻书数目：1", "highlighted": false},
        {"id": 24, "cx": 1991.54, "cy": 1400.88, "r": 10, "attributes": ["宋高宗 赵构", "绍兴"], "info": "齐安郡学\n淮南西路（路）\n黄州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 25, "cx": 1992.41, "cy": 1444.03, "r": 10, "attributes": ["宋理宗 赵顼", "宝祐"], "info": "临江郡\n淮南西路（路）\n和州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 26, "cx": 1994.76, "cy": 1535.72, "r": 10, "attributes": ["宋理宗 赵顼", "淳祐"], "info": "徽州\n江南东路（路）\n歙州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 27, "cx": 2006.59, "cy": 1556.16, "r": 10, "attributes": ["宋理宗 赵顼", "淳祐"], "info": "涂郡斋\n江南东路（路）\n太平州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 28, "cx": 2007.81, "cy": 1504.77, "r": 10, "attributes": ["宋宁宗 赵扩", "嘉泰"], "info": "秋浦郡\n江南东路（路）\n池州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 29, "cx": 2015.92, "cy": 1476.95, "r": 10, "attributes": ["宋宁宗 赵扩", "嘉定"], "info": "当涂郡\n江南东路（路）\n太平州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 31, "cx": 2025.36, "cy": 1527.36, "r": 16, "attributes": ["宋孝宗 赵眘", "淳熙","绍熙"], "info": "池阳郡\n江南东路（路）\n池州（州）\n\n刻书数目：4", "highlighted": false},
        {"id": 32, "cx": 2029.09, "cy": 1573.27, "r": 10, "attributes": ["宋理宗 赵顼", "淳祐"], "info": "上饶郡学\n江南东路（路）\n信州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 30, "cx": 2030.08, "cy": 1727.57, "r": 10, "attributes": [NaN, NaN], "info": "建安一经堂\n福建路（路）\n\n\n刻书数目：1", "highlighted": false},
        {"id": 33, "cx": 2032.37, "cy": 1371.15, "r": 12, "attributes": ["宋孝宗 赵眘","宋宁宗 赵扩", "淳熙","嘉定"], "info": "滁阳郡斋\n淮南东路（路）\n滁州（州）\n\n刻书数目：2", "highlighted": false},
        {"id": 34, "cx": 2034.13, "cy": 1551.16, "r": 10, "attributes": ["宋光宗 赵惇", "绍熙"], "info": "盱江郡\n江南西路（路）\n抚州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 35, "cx": 2039.43, "cy": 1700.7, "r": 12, "attributes": ["宋理宗 赵顼", "淳祐"], "info": "建安余卓\n福建路（路）\n建宁府（州）\n\n刻书数目：2", "highlighted": false},
        {"id": 36, "cx": 2041.78, "cy": 1436.98, "r": 14, "attributes": ["宋孝宗 赵眘", "干道"], "info": "姑孰郡\n江南东路（路）\n太平州（州）\n\n刻书数目：3", "highlighted": false},
        {"id": 37, "cx": 2050.4, "cy": 1531.54, "r": 10, "attributes": ["宋理宗 赵顼", "开庆"], "info": "太平天寿寺\n江南东路（路）\n建康府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 38, "cx": 2050.4, "cy": 1557.42, "r": 10, "attributes": ["宋高宗 赵构", "绍兴"], "info": "宛陵郡\n江南东路（路）\n宣州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 39, "cx": 2051.18, "cy": 1500.46, "r": 10, "attributes": ["宋宁宗 赵扩", "嘉定"], "info": "池州\n江南东路（路）\n池州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 41, "cx": 2055.41, "cy": 1579.94, "r": 10, "attributes": ["宋宁宗 赵扩", "庆元"], "info": "寻阳郡斋\n江南西路（路）\n江州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 40, "cx": 2055.61, "cy": 1671.86, "r": 10, "attributes": ["宋高宗 赵构", "绍兴"], "info": "建阳崇化书坊陈八郎宅\n福建路（路）\n建宁府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 42, "cx": 2058.23, "cy": 1349.99, "r": 12, "attributes": ["宋孝宗 赵眘", "干道"], "info": "高邮军学\n淮南东路（路）\n高邮军（州）\n\n刻书数目：2", "highlighted": false},
        {"id": 44, "cx": 2060.59, "cy": 1469.89, "r": 12, "attributes": ["宋高宗 赵构", "绍兴"], "info": "江南东路转运司\n江南东路（路）\n\n\n刻书数目：1", "highlighted": false},
        {"id": 43, "cx": 2063.78, "cy": 1727.21, "r": 12, "attributes": ["宋光宗 赵惇", "绍熙"], "info": "余仁仲万卷堂\n福建路（路）\n南剑州（州）\n\n刻书数目：2", "highlighted": false},
        {"id": 45, "cx": 2064.36, "cy": 1549.15, "r": 10, "attributes": ["宋宁宗 赵扩", "嘉泰"], "info": "筠阳郡斋\n江南西路（路）\n袁州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 46, "cx": 2067.64, "cy": 1392.31, "r": 12, "attributes": ["宋孝宗 赵眘", "淳熙"], "info": "桐川郡\n江南东路（路）\n建康府（州）\n\n刻书数目：2", "highlighted": false},
        {"id": 47, "cx": 2069.37, "cy": 1564.17, "r": 10, "attributes": ["宋孝宗 赵眘", "淳熙"], "info": "筠州公使库\n江南西路（路）\n袁州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 49, "cx": 2073.02, "cy": 1642.85, "r": 10, "attributes": ["宋宁宗 赵扩", "开禧"], "info": "建安刘日新宅三桂堂\n福建路（路）\n建宁府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 48, "cx": 2077.49, "cy": 1776.72, "r": 10, "attributes": ["宋孝宗 赵眘", "干道"], "info": "蔡梦弼东塾-建溪\n福建路（路）\n南剑州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 50, "cx": 2079.3, "cy": 1506.15, "r": 10, "attributes": ["宋理宗 赵顼", "嘉熙"], "info": "新定郡\n两浙西路（路）\n建德府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 51, "cx": 2084.1, "cy": 1436.98, "r": 12, "attributes": ["宋宁宗 赵扩", "嘉泰"], "info": "新安郡\n江南东路（路）\n歙州（州）\n\n刻书数目：2", "highlighted": false},
        {"id": 52, "cx": 2089.32, "cy": 1483.62, "r": 12, "attributes": ["宋孝宗 赵眘", "淳熙"], "info": "严陵郡\n两浙西路（路）\n建德府（州）\n\n刻书数目：2", "highlighted": false},
        {"id": 53, "cx": 2090.42, "cy": 1412.68, "r": 10, "attributes": ["宋理宗 赵顼", "淳祐"], "info": "袁州\n江南西路（路）\n袁州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 54, "cx": 2091.82, "cy": 1501.15, "r": 10, "attributes": ["宋高宗 赵构", "绍兴"], "info": "吉州\n两浙西路（路）\n吉州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 55, "cx": 2095.68, "cy": 1463.6, "r": 10, "attributes": ["宋孝宗 赵眘", "干道"], "info": "秀州惠云院僧德\n两浙西路（路）\n嘉兴府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 60, "cx": 2098.38, "cy": 1684.62, "r": 35, "attributes": ["宋高宗 赵构","宋孝宗 赵眘","宋理宗 赵顼", "建炎","绍兴","靖康","干道"], "info": "福州开元寺\n福建路（路）\n福州（州）\n\n刻书数目：667", "highlighted": false},
        {"id": 56, "cx": 2098.6, "cy": 1734.58, "r": 12, "attributes": ["宋理宗 赵顼", "绍定"], "info": "积德堂\n福建路（路）\n南剑州（州）\n\n刻书数目：2", "highlighted": false},
        {"id": 57, "cx": 2099.05, "cy": 1381.1, "r": 10, "attributes": ["宋宁宗 赵扩", "嘉定"], "info": "菩提教院\n两浙西路（路）\n临安府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 58, "cx": 2101.68, "cy": 1398.9, "r": 10, "attributes": ["宋宁宗 赵扩", "嘉定"], "info": "云间洞天\n两浙西路（路）\n嘉兴府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 59, "cx": 2104.36, "cy": 1626.6, "r": 14, "attributes": ["宋宁宗 赵扩","宋理宗 赵顼", "嘉定","宝庆"], "info": "建宁郡\n福建路（路）\n建宁府（州）\n\n刻书数目：3", "highlighted": false},
        {"id": 61, "cx": 2108.09, "cy": 1496.14, "r": 10, "attributes": ["宋高宗 赵构", NaN], "info": "万卷堂\n两浙西路（路）\n平江府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 62, "cx": 2109.34, "cy": 1479.87, "r": 10, "attributes": ["宋理宗 赵顼", "绍定"], "info": "临安藏书阁\n两浙西路（路）\n\n\n刻书数目：1", "highlighted": false},
        {"id": 63, "cx": 2110.32, "cy": 1367.33, "r": 10, "attributes": ["宋孝宗 赵眘", "淳熙"], "info": "镇江府学\n两浙西路（路）\n镇江府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 64, "cx": 2111.85, "cy": 1521.18, "r": 10, "attributes": ["宋孝宗 赵眘", "淳熙"], "info": "象山县学\n两浙东路（路）\n庆元府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 65, "cx": 2115.96, "cy": 1421.94, "r": 10, "attributes": ["宋孝宗 赵眘", "淳熙"], "info": "严州郡\n两浙西路（路）\n建德府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 66, "cx": 2117.51, "cy": 1456.57, "r": 10, "attributes": ["宋理宗 赵顼", "端平"], "info": "常州\n两浙西路（路）\n常州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 67, "cx": 2118.37, "cy": 1388.19, "r": 10, "attributes": ["宋理宗 赵顼", "嘉熙"], "info": "侃径山化城接待院\n两浙西路（路）\n临安府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 68, "cx": 2124.04, "cy": 1472.89, "r": 10, "attributes": ["宋高宗 赵构", "绍兴"], "info": "临安府荣六郎家\n两浙西路（路）\n临安府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 69, "cx": 2127.93, "cy": 1406.86, "r": 10, "attributes": ["宋高宗 赵构", "绍兴"], "info": "建康郡\n两浙西路（路）\n建康府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 70, "cx": 2130.35, "cy": 1372.34, "r": 10, "attributes": ["宋理宗 赵顼", "淳祐"], "info": "平江府常熟顾霆发\n两浙西路（路）\n平江府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 71, "cx": 2133.83, "cy": 1458.2, "r": 10, "attributes": ["宋宁宗 赵扩", "嘉定"], "info": "陆子遹溧阳学宫\n两浙西路（路）\n常州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 72, "cx": 2134.65, "cy": 1423.1, "r": 10, "attributes": [NaN, NaN], "info": "杭州\n两浙西路（路）\n杭州府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 73, "cx": 2135.46, "cy": 1440.24, "r": 10, "attributes": ["宋高宗 赵构", "绍兴"], "info": "湖州报恩光孝禅寺\n两浙西路（路）\n湖州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 76, "cx": 2136.68, "cy": 1643.42, "r": 45, "attributes": ["宋高宗 赵构","宋孝宗 赵眘", "建炎","绍兴","干道"], "info": "福州东禅等觉院\n福建路（路）\n福州（州）\n\n刻书数目：7", "highlighted": false},
        {"id": 78, "cx": 2137.84, "cy": 1725.82, "r": 10, "attributes": ["宋高宗 赵构", NaN], "info": "福州鼓山寺\n福建路（路）\n福州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 74, "cx": 2139.09, "cy": 1517.55, "r": 10, "attributes": ["宋宁宗 赵扩", "庆元"], "info": "绍兴府\n两浙东路（路）\n绍兴府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 75, "cx": 2139.23, "cy": 1392.95, "r": 10, "attributes": ["宋光宗 赵惇", "绍熙"], "info": "锦溪张监税\n两浙西路（路）\n平江府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 80, "cx": 2149.62, "cy": 1568.16, "r": 10, "attributes": ["宋光宗 赵惇", "绍兴"], "info": "温州州学\n两浙东路（路）\n台州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 77, "cx": 2151.19, "cy": 1486.57, "r": 10, "attributes": ["宋光宗 赵惇", "绍熙"], "info": "会稽郡斋\n两浙东路（路）\n绍兴府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 79, "cx": 2151.79, "cy": 1414.12, "r": 10, "attributes": ["宋恭宗 赵㬎", "德祐"], "info": "华亭义塾\n两浙西路（路）\n嘉兴府（州）\n华亭县（县）\n刻书数目：1", "highlighted": false},
        {"id": 82, "cx": 2156.63, "cy": 1685.42, "r": 10, "attributes": ["宋理宗 赵顼", "开庆"], "info": "福州学官\n福建路（路）\n福州（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 81, "cx": 2159.89, "cy": 1435.67, "r": 16, "attributes": ["宋高宗 赵构", "绍兴"], "info": "临安府\n两浙西路（路）\n临安府（州）\n\n刻书数目：4", "highlighted": false},
        {"id": 83, "cx": 2160.88, "cy": 1466.13, "r": 10, "attributes": [NaN, NaN], "info": NaN, "highlighted": false},
        {"id": 84, "cx": 2168.35, "cy": 1411.93, "r": 10, "attributes": ["宋宁宗 赵扩", "庆元"], "info": "华亭县\n两浙西路（路）\n嘉兴府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 85, "cx": 2174.61, "cy": 1428.2, "r": 10, "attributes": ["宋宁宗 赵扩", "庆元"], "info": "书隐斋\n两浙西路（路）\n嘉兴府（州）\n\n刻书数目：1", "highlighted": false},
        {"id": 86, "cx": 2175.56, "cy": 1387.51, "r": 40, "attributes": [NaN, NaN], "info": "碛砂延圣院\n两浙西路（路）\n平江府（州）\n\n刻书数目：1183", "highlighted": false},
        {"id": 87, "cx": 2180.2, "cy": 1483.67, "r": 10, "attributes": [NaN, NaN], "info": "两浙东路茶监司\n两浙东路（路）\n\n\n刻书数目：1", "highlighted": false},
        {"id": 88, "cx": 2182.84, "cy": 1540.84, "r": 12, "attributes": ["宋高宗 赵构", "绍兴"], "info": "两浙东路茶盐司公使库\n两浙东路（路）\n\n\n刻书数目：2", "highlighted": false},
        {"id": 89, "cx": 2184.62, "cy": 1415.69, "r": 10, "attributes": ["宋高宗 赵构", "绍兴"], "info": "南山慧因讲院\n两浙西路（路）\n临安府(州）\n\n刻书数目：1", "highlighted": false},
        {"id": 92, "cx": 2189.07, "cy": 1455.53, "r": 10, "attributes": ["宋度宗 赵禥", "咸淳"], "info": "阿育王山\n两浙东路（路）\n庆元府(州）\n\n刻书数目：1", "highlighted": false},
        {"id": 90, "cx": 2194.59, "cy": 1510.27, "r": 14, "attributes": ["宋高宗 赵构", "绍兴"], "info": "两浙东路茶盐司\n两浙东路（路）\n\n\n刻书数目：3", "highlighted": false},
        {"id": 91, "cx": 2200.2, "cy": 1405.67, "r": 10, "attributes": ["宋孝宗 赵眘", "干道"], "info": "两浙东路茶盐司\n两浙西路（路）\n平江府（州）\n\n刻书数目：1", "highlighted": false}
      ],
      //南北宋数据
      bei: [
        {
          name: '宋太祖 赵匡胤',
          reigns: [
            { label: '建隆', id: 0 },
            { label: '乾德', id: 1 },
            { label: '开宝', id: 2 },
          ],
        },
        {
          name: '宋太宗 赵光义',
          reigns: [
            { label: '太平兴国', id: 3 },
            { label: '雍熙', id: 4 },
            { label: '端拱', id: 5 },
            { label: '淳化', id: 6 },
            { label: '至道', id: 7 },
          ],
        },
        {
          name: '宋真宗 赵恒',
          reigns: [
            { label: '咸平', id: 8 },
            { label: '景德', id: 9 },
            { label: '大中祥符', id: 10 },
            { label: '天禧', id: 11 },
            { label: '乾兴', id: 12 },
          ],
        },
        {
          name: '宋仁宗 赵祯',
          reigns: [
            { label: '天圣', id: 13 },
            { label: '明道', id: 14 },
            { label: '景祐', id: 15 },
            { label: '宝元', id: 16 },
            { label: '康定', id: 17 },
            { label: '庆历', id: 18 },
            { label: '皇祐', id: 19 },
          ],
        },
        {
          name: '宋英宗 赵曙',
          reigns: [
            { label: '治平', id: 20 },
          ],
        },
        {
          name: '宋神宗 赵顼',
          reigns: [
            { label: '熙宁', id: 21 },
            { label: '元丰', id: 22 },
          ],
        },
        {
          name: '宋哲宗 赵煦',
          reigns: [
            { label: '元祐', id: 23 },
            { label: '绍圣', id: 24 },
            { label: '元符', id: 25 },
          ],
        },
        {
          name: '宋徽宗 赵佶',
          reigns: [
            { label: '建中靖国', id: 26 },
            { label: '崇宁', id: 27 },
            { label: '大观', id: 28 },
            { label: '政和', id: 29 },
            { label: '重和', id: 30 },
            { label: '宣和', id: 31 },
          ],
        },
        {
          name: '宋钦宗 赵桓',
          reigns: [
            { label: '靖康', id: 32 },
          ],
        },
      ],
      nan: [
        {
          name: '宋高宗 赵构',
          reigns: [
            { label: '建炎', id: 0 },
            { label: '绍兴', id: 1 },
          ],
        },
        {
          name: '宋孝宗 赵昚',
          reigns: [
            { label: '隆兴', id: 2 },
            { label: '乾道', id: 3 },
            { label: '淳熙', id: 4 },
          ],
        },
        {
          name: '宋光宗 赵惇',
          reigns: [
            { label: '绍熙', id: 5 },
          ],
        },
        {
          name: '宋宁宗 赵扩',
          reigns: [
            { label: '庆元', id: 6 },
            { label: '嘉泰', id: 7 },
            { label: '开禧', id: 8 },
            { label: '嘉定', id: 9 },
          ],
        },
        {
          name: '宋理宗 赵昀',
          reigns: [
            { label: '宝庆', id: 10 },
            { label: '绍定', id: 11 },
            { label: '端平', id: 12 },
            { label: '嘉熙', id: 13 },
            { label: '淳祐', id: 14 },
            { label: '宝祐', id: 15 },
            { label: '开庆', id: 16 },
            { label: '景定', id: 17 },
          ],
        },
        {
          name: '宋度宗 赵禥',
          reigns: [
            { label: '咸淳', id: 18 },
          ],
        },
      ],
      tooltip: {
        visible: false,
        x: 0,
        y: 0,
        info: ''
      },
      bars: [
        {"label":961,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":966,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":971,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":17,"color":"#994925"},{"x":17,"width":0,"color":"#F4E8D1"},{"x":17,"width":0,"color":"#B6BEA5"}]},
        {"label":972,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":973,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":19,"color":"#994925"},{"x":19,"width":0,"color":"#F4E8D1"},{"x":19,"width":0,"color":"#B6BEA5"}]},
        {"label":974,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":18,"color":"#994925"},{"x":18,"width":0,"color":"#F4E8D1"},{"x":18,"width":0,"color":"#B6BEA5"}]},
        {"label":975,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":986,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":988,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":17,"color":"#994925"},{"x":17,"width":0,"color":"#F4E8D1"},{"x":17,"width":0,"color":"#B6BEA5"}]},
        {"label":1001,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1013,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1015,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1017,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":19,"color":"#994925"},{"x":19,"width":0,"color":"#F4E8D1"},{"x":19,"width":0,"color":"#B6BEA5"}]},
        {"label":1024,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1035,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1036,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1042,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1043,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":20,"color":"#994925"},{"x":20,"width":0,"color":"#F4E8D1"},{"x":20,"width":0,"color":"#B6BEA5"}]},
        {"label":1044,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1054,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1060,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1063,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1068,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":18,"color":"#994925"},{"x":18,"width":0,"color":"#F4E8D1"},{"x":18,"width":0,"color":"#B6BEA5"}]},
        {"label":1069,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1077,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1078,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1080,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":23,"color":"#994925"},{"x":23,"width":0,"color":"#F4E8D1"},{"x":23,"width":0,"color":"#B6BEA5"}]},
        {"label":1084,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":16,"color":"#994925"},{"x":32,"width":0,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1085,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":150,"color":"#994925"},{"x":150,"width":0,"color":"#F4E8D1"},{"x":150,"width":0,"color":"#B6BEA5"}]},
        {"label":1087,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1088,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1090,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":112,"color":"#994925"},{"x":112,"width":0,"color":"#F4E8D1"},{"x":112,"width":0,"color":"#B6BEA5"}]},
        {"label":1091,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":150,"color":"#994925"},{"x":150,"width":0,"color":"#F4E8D1"},{"x":150,"width":0,"color":"#B6BEA5"}]},
        {"label":1093,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":41,"color":"#994925"},{"x":41,"width":0,"color":"#F4E8D1"},{"x":41,"width":0,"color":"#B6BEA5"}]},
        {"label":1094,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":55,"color":"#994925"},{"x":55,"width":0,"color":"#F4E8D1"},{"x":55,"width":0,"color":"#B6BEA5"}]},
        {"label":1095,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":48,"color":"#994925"},{"x":48,"width":0,"color":"#F4E8D1"},{"x":48,"width":0,"color":"#B6BEA5"}]},
        {"label":1096,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":36,"color":"#994925"},{"x":36,"width":0,"color":"#F4E8D1"},{"x":36,"width":0,"color":"#B6BEA5"}]},
        {"label":1097,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":150,"color":"#994925"},{"x":150,"width":0,"color":"#F4E8D1"},{"x":150,"width":0,"color":"#B6BEA5"}]},
        {"label":1098,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":25,"color":"#994925"},{"x":25,"width":0,"color":"#F4E8D1"},{"x":25,"width":0,"color":"#B6BEA5"}]},
        {"label":1099,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1100,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":71,"color":"#994925"},{"x":71,"width":0,"color":"#F4E8D1"},{"x":71,"width":0,"color":"#B6BEA5"}]},
        {"label":1101,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":26,"color":"#994925"},{"x":26,"width":0,"color":"#F4E8D1"},{"x":26,"width":0,"color":"#B6BEA5"}]},
        {"label":1102,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":88,"color":"#994925"},{"x":104,"width":0,"color":"#F4E8D1"},{"x":104,"width":0,"color":"#B6BEA5"}]},
        {"label":1103,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":30,"color":"#994925"},{"x":46,"width":0,"color":"#F4E8D1"},{"x":46,"width":0,"color":"#B6BEA5"}]},
        {"label":1104,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":32,"color":"#994925"},{"x":32,"width":0,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1105,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1106,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":70,"color":"#994925"},{"x":70,"width":0,"color":"#F4E8D1"},{"x":70,"width":0,"color":"#B6BEA5"}]},
        {"label":1107,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":136,"color":"#994925"},{"x":136,"width":0,"color":"#F4E8D1"},{"x":136,"width":0,"color":"#B6BEA5"}]},
        {"label":1108,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1109,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":40,"color":"#994925"},{"x":40,"width":0,"color":"#F4E8D1"},{"x":40,"width":0,"color":"#B6BEA5"}]},
        {"label":1110,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":55,"color":"#994925"},{"x":55,"width":0,"color":"#F4E8D1"},{"x":55,"width":0,"color":"#B6BEA5"}]},
        {"label":1111,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":82,"color":"#994925"},{"x":82,"width":0,"color":"#F4E8D1"},{"x":82,"width":0,"color":"#B6BEA5"}]},
        {"label":1112,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":39,"color":"#994925"},{"x":39,"width":0,"color":"#F4E8D1"},{"x":39,"width":0,"color":"#B6BEA5"}]},
        {"label":1113,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1117,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":27,"color":"#994925"},{"x":27,"width":0,"color":"#F4E8D1"},{"x":27,"width":0,"color":"#B6BEA5"}]},
        {"label":1118,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":19,"color":"#994925"},{"x":19,"width":0,"color":"#F4E8D1"},{"x":19,"width":0,"color":"#B6BEA5"}]},
        {"label":1119,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1124,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":61,"color":"#994925"},{"x":77,"width":0,"color":"#F4E8D1"},{"x":77,"width":0,"color":"#B6BEA5"}]},
        {"label":1125,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":54,"color":"#994925"},{"x":54,"width":0,"color":"#F4E8D1"},{"x":54,"width":0,"color":"#B6BEA5"}]},
        {"label":1126,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":150,"color":"#994925"},{"x":150,"width":0,"color":"#F4E8D1"},{"x":150,"width":0,"color":"#B6BEA5"}]},
        {"label":1127,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":54,"color":"#994925"},{"x":54,"width":0,"color":"#F4E8D1"},{"x":54,"width":0,"color":"#B6BEA5"}]},
        {"label":1128,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":61,"color":"#994925"},{"x":61,"width":0,"color":"#F4E8D1"},{"x":61,"width":0,"color":"#B6BEA5"}]},
        {"label":1129,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1131,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":19,"color":"#994925"},{"x":19,"width":0,"color":"#F4E8D1"},{"x":19,"width":0,"color":"#B6BEA5"}]},
        {"label":1132,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":19,"color":"#E4C58E"},{"x":19,"width":16,"color":"#994925"},{"x":35,"width":0,"color":"#F4E8D1"},{"x":35,"width":0,"color":"#B6BEA5"}]},
        {"label":1133,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":21,"color":"#994925"},{"x":21,"width":0,"color":"#F4E8D1"},{"x":21,"width":0,"color":"#B6BEA5"}]},
        {"label":1134,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":18,"color":"#E4C58E"},{"x":18,"width":29,"color":"#994925"},{"x":47,"width":0,"color":"#F4E8D1"},{"x":47,"width":0,"color":"#B6BEA5"}]},
        {"label":1135,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1138,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":27,"color":"#994925"},{"x":27,"width":0,"color":"#F4E8D1"},{"x":27,"width":0,"color":"#B6BEA5"}]},
        {"label":1139,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":0,"color":"#E4C58E"},{"x":16,"width":17,"color":"#994925"},{"x":33,"width":16,"color":"#F4E8D1"},{"x":49,"width":0,"color":"#B6BEA5"}]},
        {"label":1140,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":17,"color":"#F4E8D1"},{"x":17,"width":0,"color":"#B6BEA5"}]},
        {"label":1141,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1142,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1143,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":16,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1145,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":0,"color":"#E4C58E"},{"x":16,"width":16,"color":"#994925"},{"x":32,"width":0,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1146,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":25,"color":"#994925"},{"x":25,"width":0,"color":"#F4E8D1"},{"x":25,"width":0,"color":"#B6BEA5"}]},
        {"label":1147,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":16,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1148,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":150,"color":"#994925"},{"x":166,"width":16,"color":"#F4E8D1"},{"x":182,"width":0,"color":"#B6BEA5"}]},
        {"label":1149,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":23,"color":"#994925"},{"x":23,"width":0,"color":"#F4E8D1"},{"x":23,"width":0,"color":"#B6BEA5"}]},
        {"label":1150,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":19,"color":"#994925"},{"x":19,"width":16,"color":"#F4E8D1"},{"x":35,"width":0,"color":"#B6BEA5"}]},
        {"label":1151,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1152,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":17,"color":"#994925"},{"x":17,"width":0,"color":"#F4E8D1"},{"x":17,"width":0,"color":"#B6BEA5"}]},
        {"label":1154,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1158,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1160,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":16,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1161,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":18,"color":"#994925"},{"x":18,"width":16,"color":"#F4E8D1"},{"x":34,"width":0,"color":"#B6BEA5"}]},
        {"label":1162,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1165,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":16,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1166,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":16,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1167,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":0,"color":"#E4C58E"},{"x":16,"width":16,"color":"#994925"},{"x":32,"width":0,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1169,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":0,"color":"#E4C58E"},{"x":16,"width":16,"color":"#994925"},{"x":32,"width":0,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1170,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1171,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":17,"color":"#994925"},{"x":33,"width":16,"color":"#F4E8D1"},{"x":49,"width":0,"color":"#B6BEA5"}]},
        {"label":1172,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":17,"color":"#994925"},{"x":33,"width":16,"color":"#F4E8D1"},{"x":49,"width":0,"color":"#B6BEA5"}]},
        {"label":1173,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":18,"color":"#F4E8D1"},{"x":34,"width":0,"color":"#B6BEA5"}]},
        {"label":1175,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":17,"color":"#E4C58E"},{"x":33,"width":0,"color":"#994925"},{"x":33,"width":0,"color":"#F4E8D1"},{"x":33,"width":0,"color":"#B6BEA5"}]},
        {"label":1176,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":17,"color":"#E4C58E"},{"x":33,"width":0,"color":"#994925"},{"x":33,"width":0,"color":"#F4E8D1"},{"x":33,"width":0,"color":"#B6BEA5"}]},
        {"label":1177,"segments":[{"x":0,"width":18,"color":"#C9C990"},{"x":18,"width":0,"color":"#E4C58E"},{"x":18,"width":0,"color":"#994925"},{"x":18,"width":0,"color":"#F4E8D1"},{"x":18,"width":0,"color":"#B6BEA5"}]},
        {"label":1178,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":16,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1179,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1180,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":0,"color":"#E4C58E"},{"x":16,"width":16,"color":"#994925"},{"x":32,"width":0,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1181,"segments":[{"x":0,"width":18,"color":"#C9C990"},{"x":18,"width":0,"color":"#E4C58E"},{"x":18,"width":16,"color":"#994925"},{"x":34,"width":17,"color":"#F4E8D1"},{"x":51,"width":0,"color":"#B6BEA5"}]},
        {"label":1182,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":0,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1183,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1184,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1185,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1186,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1187,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":16,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1190,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":16,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1191,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":16,"color":"#E4C58E"},{"x":32,"width":0,"color":"#994925"},{"x":32,"width":0,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1192,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":16,"color":"#E4C58E"},{"x":32,"width":0,"color":"#994925"},{"x":32,"width":16,"color":"#F4E8D1"},{"x":48,"width":0,"color":"#B6BEA5"}]},
        {"label":1193,"segments":[{"x":0,"width":17,"color":"#C9C990"},{"x":17,"width":0,"color":"#E4C58E"},{"x":17,"width":0,"color":"#994925"},{"x":17,"width":16,"color":"#F4E8D1"},{"x":33,"width":0,"color":"#B6BEA5"}]},
        {"label":1194,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":17,"color":"#E4C58E"},{"x":17,"width":0,"color":"#994925"},{"x":17,"width":0,"color":"#F4E8D1"},{"x":17,"width":0,"color":"#B6BEA5"}]},
        {"label":1195,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1196,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":16,"color":"#F4E8D1"},{"x":16,"width":16,"color":"#B6BEA5"}]},
        {"label":1197,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":16,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1199,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":17,"color":"#F4E8D1"},{"x":17,"width":0,"color":"#B6BEA5"}]},
        {"label":1200,"segments":[{"x":0,"width":17,"color":"#C9C990"},{"x":17,"width":0,"color":"#E4C58E"},{"x":17,"width":0,"color":"#994925"},{"x":17,"width":18,"color":"#F4E8D1"},{"x":35,"width":0,"color":"#B6BEA5"}]},
        {"label":1201,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":17,"color":"#F4E8D1"},{"x":33,"width":0,"color":"#B6BEA5"}]},
        {"label":1202,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1204,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":16,"color":"#994925"},{"x":32,"width":17,"color":"#F4E8D1"},{"x":49,"width":0,"color":"#B6BEA5"}]},
        {"label":1205,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":0,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1206,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":16,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1207,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":16,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1210,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":17,"color":"#994925"},{"x":33,"width":0,"color":"#F4E8D1"},{"x":33,"width":0,"color":"#B6BEA5"}]},
        {"label":1211,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":16,"color":"#E4C58E"},{"x":32,"width":16,"color":"#994925"},{"x":48,"width":16,"color":"#F4E8D1"},{"x":64,"width":0,"color":"#B6BEA5"}]},
        {"label":1212,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":17,"color":"#994925"},{"x":17,"width":16,"color":"#F4E8D1"},{"x":33,"width":0,"color":"#B6BEA5"}]},
        {"label":1213,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":0,"color":"#E4C58E"},{"x":16,"width":16,"color":"#994925"},{"x":32,"width":16,"color":"#F4E8D1"},{"x":48,"width":0,"color":"#B6BEA5"}]},
        {"label":1215,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1216,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":0,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1217,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":0,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1218,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1220,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":16,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1221,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":16,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1223,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":0,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1225,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":16,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1226,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":16,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1228,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":16,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1229,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":17,"color":"#E4C58E"},{"x":17,"width":18,"color":"#994925"},{"x":35,"width":16,"color":"#F4E8D1"},{"x":51,"width":0,"color":"#B6BEA5"}]},
        {"label":1230,"segments":[{"x":0,"width":18,"color":"#C9C990"},{"x":18,"width":0,"color":"#E4C58E"},{"x":18,"width":0,"color":"#994925"},{"x":18,"width":0,"color":"#F4E8D1"},{"x":18,"width":0,"color":"#B6BEA5"}]},
        {"label":1232,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":16,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1233,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":16,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1234,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":19,"color":"#994925"},{"x":19,"width":0,"color":"#F4E8D1"},{"x":19,"width":0,"color":"#B6BEA5"}]},
        {"label":1235,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":17,"color":"#994925"},{"x":17,"width":0,"color":"#F4E8D1"},{"x":17,"width":0,"color":"#B6BEA5"}]},
        {"label":1236,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":16,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1238,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":38,"color":"#994925"},{"x":38,"width":0,"color":"#F4E8D1"},{"x":38,"width":0,"color":"#B6BEA5"}]},
        {"label":1239,"segments":[{"x":0,"width":17,"color":"#C9C990"},{"x":17,"width":16,"color":"#E4C58E"},{"x":33,"width":57,"color":"#994925"},{"x":90,"width":0,"color":"#F4E8D1"},{"x":90,"width":0,"color":"#B6BEA5"}]},
        {"label":1240,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":16,"color":"#E4C58E"},{"x":32,"width":32,"color":"#994925"},{"x":64,"width":0,"color":"#F4E8D1"},{"x":64,"width":0,"color":"#B6BEA5"}]},
        {"label":1241,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":32,"color":"#994925"},{"x":32,"width":16,"color":"#F4E8D1"},{"x":48,"width":0,"color":"#B6BEA5"}]},
        {"label":1242,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1243,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":28,"color":"#994925"},{"x":28,"width":16,"color":"#F4E8D1"},{"x":44,"width":0,"color":"#B6BEA5"}]},
        {"label":1244,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":17,"color":"#994925"},{"x":17,"width":0,"color":"#F4E8D1"},{"x":17,"width":0,"color":"#B6BEA5"}]},
        {"label":1245,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":19,"color":"#994925"},{"x":19,"width":0,"color":"#F4E8D1"},{"x":19,"width":0,"color":"#B6BEA5"}]},
        {"label":1246,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":0,"color":"#E4C58E"},{"x":16,"width":17,"color":"#994925"},{"x":33,"width":0,"color":"#F4E8D1"},{"x":33,"width":0,"color":"#B6BEA5"}]},
        {"label":1248,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":17,"color":"#994925"},{"x":17,"width":0,"color":"#F4E8D1"},{"x":17,"width":0,"color":"#B6BEA5"}]},
        {"label":1249,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":18,"color":"#994925"},{"x":34,"width":16,"color":"#F4E8D1"},{"x":50,"width":0,"color":"#B6BEA5"}]},
        {"label":1250,"segments":[{"x":0,"width":17,"color":"#C9C990"},{"x":17,"width":16,"color":"#E4C58E"},{"x":33,"width":0,"color":"#994925"},{"x":33,"width":0,"color":"#F4E8D1"},{"x":33,"width":0,"color":"#B6BEA5"}]},
        {"label":1252,"segments":[{"x":0,"width":20,"color":"#C9C990"},{"x":20,"width":0,"color":"#E4C58E"},{"x":20,"width":0,"color":"#994925"},{"x":20,"width":0,"color":"#F4E8D1"},{"x":20,"width":0,"color":"#B6BEA5"}]},
        {"label":1255,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":0,"color":"#E4C58E"},{"x":16,"width":17,"color":"#994925"},{"x":33,"width":0,"color":"#F4E8D1"},{"x":33,"width":0,"color":"#B6BEA5"}]},
        {"label":1256,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":16,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1257,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":18,"color":"#E4C58E"},{"x":18,"width":0,"color":"#994925"},{"x":18,"width":0,"color":"#F4E8D1"},{"x":18,"width":0,"color":"#B6BEA5"}]},
        {"label":1259,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":17,"color":"#994925"},{"x":33,"width":16,"color":"#F4E8D1"},{"x":49,"width":0,"color":"#B6BEA5"}]},
        {"label":1260,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1261,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1262,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1264,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":22,"color":"#994925"},{"x":22,"width":0,"color":"#F4E8D1"},{"x":22,"width":0,"color":"#B6BEA5"}]},
        {"label":1265,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":16,"color":"#E4C58E"},{"x":32,"width":18,"color":"#994925"},{"x":50,"width":0,"color":"#F4E8D1"},{"x":50,"width":0,"color":"#B6BEA5"}]},
        {"label":1266,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":20,"color":"#994925"},{"x":20,"width":0,"color":"#F4E8D1"},{"x":20,"width":0,"color":"#B6BEA5"}]},
        {"label":1267,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":17,"color":"#994925"},{"x":33,"width":0,"color":"#F4E8D1"},{"x":33,"width":0,"color":"#B6BEA5"}]},
        {"label":1268,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1269,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":16,"color":"#F4E8D1"},{"x":32,"width":0,"color":"#B6BEA5"}]},
        {"label":1273,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":16,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1275,"segments":[{"x":0,"width":16,"color":"#C9C990"},{"x":16,"width":0,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1303,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":0,"color":"#994925"},{"x":0,"width":16,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1305,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":0,"color":"#E4C58E"},{"x":0,"width":16,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]},
        {"label":1335,"segments":[{"x":0,"width":0,"color":"#C9C990"},{"x":0,"width":16,"color":"#E4C58E"},{"x":16,"width":0,"color":"#994925"},{"x":16,"width":0,"color":"#F4E8D1"},{"x":16,"width":0,"color":"#B6BEA5"}]}
      ],
      // 地图放大参数
      scale: 0.5,
      zoomSpeed: 0.1,
      initialScale: 1,
      // 拖动参数
      isDragging: false,
      startX: 0,
      startY: 0,
      translateX: 0,
      translateY: 0,
      currentX: 0,
      currentY: 0,
    };
  },

  mounted() {
    this.$refs.mapSvg.addEventListener('wheel', this.handleZoom);
    this.$refs.audio.play();
  },

  created() {
    this.loadData();
  },

  computed: {
    computedViewBox() {
      return this.showBei ? '0 0 2097 1710' : '0 0 2652 2162';
    },
    computedHref() {
      return this.showBei ? '../public/insert-img/北宋地图.svg' : '../public/insert-img/南宋地图.svg';
    },
    computedCircles() {
      return this.showBei ? this.circlesbei : this.circlesnan;
    },
    computedClass() {
      return this.showBei ? 'right north' : 'right south';
    },
  },

  methods: {
    // 重置页面
    resetPage(showBei) {
      this.showBei = showBei;
      this.selectedItems = [];
      this.selectedNianhao = [];
    },

    // 播放器
    togglePlay() {
      const audio = this.$refs.audio;
      if (audio.paused) {
        audio.play();
      } else {
        audio.pause();
      }
    },

    // 更新年号
    updateNianhao(emperor) {
      const reigns = emperor.reigns;
      const isSelected = this.selectedEmperors.includes(emperor.name);
      
      if (isSelected) {
        // 添加年号
        reigns.forEach((reign) => {
          if (!this.displayedNianhao.includes(reign)) {
            this.displayedNianhao.push(reign);
          }
        });
      } else {
        // 移除年号
        this.displayedNianhao = this.displayedNianhao.filter((reign) => !reigns.includes(reign));
        // 移除相应的选中的年号
        this.selectedYears = this.selectedYears.filter((year) => !reigns.some((reign) => reign.label === year));
      }
      if(this.bei.some(e => e.name === emperor.name)){
        this.circlesbei.forEach(circle => {
        // 检查是否应该高亮
        let shouldBeHighlighted = false;
        if (this.selectedEmperors.length === 0) {
          shouldBeHighlighted = false; // 没有选择项
        } else {
          // 判断是否有任何一个选中的选项符合圆点的属性
          shouldBeHighlighted = this.selectedEmperors.some(item => circle.attributes.includes(item));
        }
        // 更新高亮状态
        circle.highlighted = shouldBeHighlighted;
        })
      }else{
        this.circlesnan.forEach(circle => {
        // 检查是否应该高亮
        let shouldBeHighlighted = false;
        if (this.selectedEmperors.length === 0) {
          shouldBeHighlighted = false; // 没有选择项
        } else {
          // 判断是否有任何一个选中的选项符合圆点的属性
          shouldBeHighlighted = this.selectedEmperors.some(item => circle.attributes.includes(item));
        }
        // 更新高亮状态
        
        circle.highlighted = shouldBeHighlighted;
      })
    }
      
    },
    updateYears(circles, item) {
      console.log(this.emperor)
      circles.forEach(circle => {
        let shouldBeHighlighted = false;

        if (this.selectedEmperors.length === 0) {
          shouldBeHighlighted = false; // 没有选择项
        } else {
          const emperorMatch = this.selectedEmperors.some(emperor => circle.attributes.includes(emperor));
          const yearMatch = this.selectedYears.some(year => circle.attributes.includes(year));

          if (this.selectedYears.length === 0) {
            shouldBeHighlighted = emperorMatch;
          } else {
            shouldBeHighlighted = emperorMatch && yearMatch;
          }
        }

        circle.highlighted = shouldBeHighlighted;
      });
    },

    // 地图展示
    updateCircles(circles) {
      // 遍历所有圆点
      circles.forEach(circle => {
        // 检查是否应该高亮
        let shouldBeHighlighted = false;
        if (this.selectedItems.length === 0) {
          shouldBeHighlighted = false; // 如果没有选择项，则全部高亮
        } else {
          // 判断是否有任何一个选中的选项符合圆点的属性
          shouldBeHighlighted = this.selectedItems.some(item => circle.attributes.includes(item));
        }
        // 更新高亮状态
        circle.highlighted = shouldBeHighlighted;
      });
    },
    showTooltip(point) {
      this.tooltip.info = point.info;
      this.tooltip.visible = true;
    },
    hideTooltip() {
      this.tooltip.visible = false;
    },
    updateTooltipPosition(event) {
      if (this.tooltip.visible) {
        const svgRect = event.target.closest('svg').getBoundingClientRect();
        console.log('svgRect:', svgRect);
        this.tooltip.x = event.clientX - svgRect.left + 10;
        this.tooltip.y = event.clientY - svgRect.top + 10;
        console.log('tooltip.x:', this.tooltip.x, 'tooltip.y:', this.tooltip.y);
      }
    },
    // 地图缩放
    handleZoom(event) {
      event.preventDefault();

      if (event.deltaY < 0) {
        // 放大
        this.scale += this.zoomSpeed;
      } else {
        // 缩小
        this.scale -= this.zoomSpeed;
      }

      // 限制缩放范围
      this.scale = Math.max(this.initialScale, this.scale);

      // 确保缩放时保持在边界内
      this.checkBounds();

      this.updateTransform();
    },
    // 开始拖动
    startDrag(event) {
      this.isDragging = true;
      this.startX = event.clientX - this.translateX;
      this.startY = event.clientY - this.translateY;
      this.$el.querySelector('.svg-container').classList.add('grabbing');
    },
    // 结束拖动
    endDrag() {
      this.isDragging = false;
      this.$el.querySelector('.svg-container').classList.remove('grabbing');
    },
    // 拖动事件
    drag(event) {
      if (this.isDragging) {
        this.translateX = event.clientX - this.startX;
        this.translateY = event.clientY - this.startY;

        // 确保拖动时保持在边界内
        this.checkBounds();

        this.updateTransform();
      }
    },
    // 检查边界
    checkBounds() {
      const container = this.$refs.mapSvg.parentElement;
      const containerWidth = container.clientWidth;
      const containerHeight = container.clientHeight;
      const mapWidth = this.$refs.mapSvg.clientWidth * this.scale;
      const mapHeight = this.$refs.mapSvg.clientHeight * this.scale;

      // 计算可拖动的最大偏移量
      const maxTranslateX = (mapWidth - containerWidth) / 2;
      const minTranslateX = -maxTranslateX;
      const maxTranslateY = (mapHeight - containerHeight) / 2;
      const minTranslateY = -maxTranslateY;

      // 限制 translateX 和 translateY 的范围
      this.translateX = Math.min(maxTranslateX, Math.max(minTranslateX, this.translateX));
      this.translateY = Math.min(maxTranslateY, Math.max(minTranslateY, this.translateY));
    },
    // 更新 transform 样式
    updateTransform() {
      this.$refs.mapSvg.style.transform = `translate(${this.translateX}px, ${this.translateY}px) scale(${this.scale})`;
    },
    // 加载数据
    async loadData() {
      try {
        const response = await fetch('dataset/output.jsonl');
        const text = await response.text();

        const data = text.split('\n').filter(line => line.trim()).map(line => JSON.parse(line));

        const categorizedData = {};

        data.forEach(item => {
          const year = item["年份"];
          const category = item["一级类目"];

          if (!categorizedData[year]) {
            categorizedData[year] = { "经": 0, "史": 0, "子": 0, "集": 0, "丛": 0};
          }

          if (!categorizedData[year][category]) {
            categorizedData[year][category] = 0;
          }

          categorizedData[year][category] += 1;
        });

        this.chartData = Object.keys(categorizedData).map(year => {
          return {
            name: year,
            ...categorizedData[year]
          };
        });

        console.log(this.chartData);
      } catch (error) {
        console.error("Error loading data:", error);
      }
    }
    
  }
};
</script>

<style>
body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}

.layout {
  background-image: url('./public/insert-img/背景图.svg');
  background-size: 100%;
  background-repeat: no-repeat;
  background-position: 0 0px;
  margin: 0;
  padding: 0;
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
}

.head {
  background-image: url('./public/insert-img/顶部导航栏.svg');
  background-size: 105%;
  background-repeat: no-repeat;
  background-position: -30px -25px;
  height: 50px;
  width: 100%;
  background-color: transparent;
  display: flex; 
  align-items: center; 
  justify-content: space-between;
  z-index: 2;
}
.tittle {
  display: flex;
  align-items: flex-end;
  color: white;
  width: 100%;
  height: 100%;
}

/* 朝代按钮 */
.switch{
  height: 100%;
  display: flex;
  align-items: end;
  margin-right: 18px;
  background-color: transparent; 
  cursor: pointer;
}
.in {
  height: 100%;
  width: 105px;
  flex: 1; /* 添加这一行 */
  text-align: center; /* 文字居中 */
  display: flex; /* 添加这一行 */
  align-items: center;
  justify-content: center;
}
.bei {
  background-image: url('./public/insert-img/北宋未点击态.svg');
  background-size: 100%;
  background-repeat: no-repeat;
  background-position: 0 3px;
  margin-right: 30px;
}
.bei:active {
  background-image: url('./public/insert-img/北宋点击态.svg');
  background-size: 100%;
  background-repeat: no-repeat;
  background-position: 0 3px;
  margin-right: 30px;
}
.nan {
  background-image: url('./public/insert-img/南宋未点击态.svg');
  background-size: 100%;
  background-repeat: no-repeat;
  background-position: 0 3px;
  margin-left: 30px;
  margin-right: 30px;
}
.nan:active{
  background-image: url('./public/insert-img/南宋点击态.svg');
  background-size: 100%;
  background-repeat: no-repeat;
  background-position: 0 3px;
  margin-left: 30px;
  margin-right: 30px;
}

/* 音乐按钮 */
.radio-button {
  position: relative;
  padding-left: 35px;
  cursor: pointer;
  font-size: 22px;
  user-select: none;
}
.radio-input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
}
.radio-label:before {
  content: "";
  position: absolute;
  left: 0;
  width: 25px;
  height: 25px;
  border: 2px solid #555;
  border-radius: 0; 
}
.radio-input:checked ~ .radio-label:before {
  background-color: #555;
}

.container {
  flex: 1;
  position: relative;
}

.left, .right {
  position: absolute;
  background-color: #A78555;
  padding: 10px;
  z-index: 2;
}

/* 左悬浮框 */
.left {
  display: flex;
  font-family: "Song";
  justify-content: space-between;
  margin-top: 15px;
  margin-left: 15px;
  width: 320px;
  height: 600px;
  overflow: hidden;
}
.selectedframe {
  background-color: white;
  width: 120px;
  height: 99%;
  overflow: hidden;
}
.lefttittle {
  width: 100%;
  height:6%;
}
.lefticon1{
  background-image: url('./public/insert-img/庙号icon.svg');
  background-size: 102%;
  background-repeat: no-repeat;
  background-position: -1px 0px;
}
.lefticon2{
  background-image: url('./public/insert-img/年号icon.svg');
  background-size: 102%;
  background-repeat: no-repeat;
  background-position: -1px 0px;
}

/* 图表 */
.chatoutlook{
  width: 180px;
  height: 99%;
  overflow: hidden;
  background-color: transparent;
}
.chart{
  width: 100%;
  height: 96%;
  margin-bottom: 2px;
  overflow: auto;
  background-color: transparent;
}
.chart::-webkit-scrollbar {
  width: 0; /* 设置滚动条的宽度为 0，使其不可见 */
}
/* 图例 */
.legend {
  display: flex;
  flex-direction: row;
  justify-content: center;
  margin-top: 12px;
}

.legend-item {
  font-size: 10px;
  display: flex;
  align-items: center;
  color: #2C3542;
  margin-right: 15px;
}

.legend-color {
  width: 5px;
  height: 5px;
  display: inline-block;
}

.legend-label {
  white-space: nowr ap;
}

/* 列表 */
.box {
  width: 100%; /* 设置宽度 */
  height: 250px; /* 设置高度 */
  overflow-y: auto; /* 当内容超出高度时显示滚动条 */
}
.box::-webkit-scrollbar {
  width: 0; /* 设置滚动条的宽度为 0，使其不可见 */
}
.box.up {
    width: 100%;
    background-color: #fff;
    border-radius: 8px;
    padding: 2px 0;
    margin: 2px 0px;
    text-align: left;
}
.list-item {
    display: flex;
    align-items: center;
    padding: 2px;
    color: #A03F29;
    border-bottom: 1px solid #A03F29;
}
.list-item:last-child {
    border-bottom: none;
}
.list-item input[type="checkbox"] {
    margin-right: 10px;
    width: 20px;
    height: 20px;
    accent-color: #A03F29;
}
.list-item label {
    flex-grow: 1;
    text-align: left;
}

/* 地图 */
.map{
  position: absolute;
  width: 100%;
  top: -500px;
  right: 220px;
  z-index: 1;
}
.svg-container {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
}
.svg-content {
  max-width: 100%;
  max-height: 100%;
  transition: transform 0.3s ease;
}
circle {
  fill: url(#highlightGradient);
  transition: fill 0.3s ease;
  cursor: pointer;
}
.highlighted {
  /* fill: blue; */
  fill: #994925;
  
}

/* 右悬浮框 */
.right {
  top: 20px;
  right: -100px;
  width: 429px;
  height: 798px;
}
.north{
  background-image: url('./public/insert-img/北宋右边固定属性图片.svg');
  background-size: 70%;
  background-repeat: no-repeat;
  background-position: -1px 0px;
  background-color: transparent;
}
.south{
  background-image: url('./public/insert-img/右边固定图片属性-南宋.svg');
  background-size: 70%;
  background-repeat: no-repeat;
  background-position: -1px 0px;
  background-color: transparent;
}
</style>