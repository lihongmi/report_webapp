<template>
  <div class="selectdorm page_content">
    <t-nav title="选择床位"></t-nav>
    <crumb :nav-list="navList"></crumb>
    <div class="selectdorm_body">
      <header>
        {{ selDromBedInfo.dorm_num }}
      </header>
      <section>
        <div class="beds">
         <ul>
        <template v-for="bed in selDromBedInfo.galleryful">
          <template v-if="$index === 3">
            <div class="aisle">
              <span class="balcony">门</span>
              <span class="top">过</span>
              <span class="bottom">道</span>
              <span class="door">阳台</span>
            </div>
          </template>
          <template v-if="!isOccupied($index + 1)">
            <li :class="{'actived': selectedIndex === $index}" @click="selectBad($index)" class="waves-light waves-effect">
              <i class="iconfont icon-chuang weixuan"></i>
              <span v-if="selectedIndex != $index">(空)</span>
              <div class="bed_num">{{$index+1}}号床</div>
            </li>
          </template>
          <template v-else>
            <li class="waves-light waves-effect">
              <i class="iconfont icon-chuang youren"></i>
              <span>已被占领 ({{isOccupied($index + 1).student.student_name | cutString 4}})</span>
              <div class="bed_num">{{$index+1}}号床</div>
              <p class="occupyTime">{{isOccupied($index + 1).created_at | fromNow}}</p>
            </li>
          </template>
        </template>
         </ul>
        </div>
      </section>
    </div>
    <div class="primary">
      <x-button :class="[submit_btn, {disabled: selectedIndex == null }]" :text="submitBtn.title" :disabled="submitBtn.disabled" @click="processButton()" type="primary"></x-button>
    </div>
    <t-footer></t-footer>
  </div>
</template>

<script>
import { XHeader, XButton } from 'vux/src/components/'
import TNav from './components/TNav'
import Crumb from './components/Crumb'
import store from './store'
import TFooter from './components/TFooter'
export default {
  components: {
    XHeader, TNav, Crumb, XButton, TFooter
  },
  data () {
    return {
      selectedIndex: null,
      // 提交按钮
      submitBtn: {
        title: '确定床位',
        disabled: true
      },
      selDromBedInfo: {},
      // 面包屑导航
      navList: [
        {title: '选择宿舍', href: 'selectdorm'}
      ]
    }
  },
  methods: {
    selectBad ($index) {
      this.selectedIndex = $index
      this.submitBtn.disabled = false
    },
    /**
     * [isOccupied 判断床位是否被占领]
     * @param bedId {Number} [床位id]
     * @return {Boolean | Object} [如果没被占领返回false 否则返回该床位信息]
     */
    isOccupied (bedId) {
      let bed = false
      this.selDromBedInfo.selected_beds.forEach(item => {
        if (item.bed_num === bedId) {
          bed = item
          return false
        }
      })
      return bed
    },
    // 点击提交按钮
    processButton () {
      this.submitBtn.title = '选择床位中'
      this.submitBtn.disabled = true
      this.$root.showConfirm('你真的真的确定选这个宿舍？选定后可在' + this.$root.modifyInfoAddr + '修改。', () => {
        store.selBed(this.selDromBedInfo.id, this.selectedIndex + 1, this).then(res => {
          // onConfirm
          // 选择宿舍成功跳转到最终页面
          this.$route.router.go('final')
        }, res => {
        })
      }, () => {
        // onCancel
        this.submitBtn.title = '确定床位 '
        this.submitBtn.disabled = false
      }, '我已下定决心!', '再想想')
    }
  },
  route: {
    data (transition) {
      // 获取当前选择的宿舍信息
      this.selDromBedInfo = JSON.parse(window.localStorage.selDromBedInfo)
      this.navList.push({
        title: this.selDromBedInfo.dorm_num
      })
      if (transition.from.name !== 'selectdorm') {
        store.getDormBeds(this.selDromBedInfo.id, this).then(res => {
          this.selDromBedInfo = res
        })
      }
    }
  }
}
</script>

<style lang="less" scoped>
.weixuan{color:#56ABE4;}
.youren{color:#A9B7B7;}
.xuanzhong{color:#ff6633;}
.selectdorm{
  background-color: #f5f5f5;
  >.selectdorm_body{
    margin-top: .5rem;
    overflow:hidden;
    >header{
        color:#999;
        font-size: .6rem;
        text-align: center;
        padding:.5rem 0;
        background-color: #fff;
        margin-bottom: .5rem;
      }
    >section{
      position: relative;
      overflow: hidden;
      .aisle{
        background-color: #f5f5f5;
        position: absolute;
        width: 20%;
        top:0;
        left:50%;
        transform: translate(-50%,0);
        height: 100%;
        >span{
          position: absolute;
        }
        >.top{
          top:20%;
          left:50%;
          transform: translate(-50%,0);
          font-size: 1rem;
        }
        >.bottom{
          bottom:20%;
          left:50%;
          transform: translate(-50%,0);
          font-size: 1rem;
        }
        >.balcony{
          top:0;
          left:50%;
          transform: translate(-50%,0);
          font-size: .6rem;
          color:#666;
        }
        >.door{
          bottom:0;
          left:50%;
          transform: translate(-50%,0);
          font-size: .6rem;
          color:#666;
        }
      }
      >.beds{
        >ul{
          background-color: #fff;
          overflow: hidden;
          >li{
            height: 5rem;
            position: relative;
            width: 40%;
            >i{
              font-size: 2.7em;
              position: absolute;
              top: 50%;
              left: 50%;
              transform: translate(-50%,-50%);
              transition: color .3s;
            }
            &:nth-of-type(even){
              float:left;
              &:not(last-child){
                border-bottom: 1px solid #f5f5f5;
              }
            }
            &:nth-of-type(odd){
              float:right;
              &:not(last-child){
                border-bottom: 1px solid #f5f5f5;
              }
            }
            &.actived{
              background-color: #eaeaea;
              >i{
                color:#FF4040;
              }
            }
            >span{
              font-size: .6rem;
              position: absolute;
              color: #666;
              bottom: .1rem;
              left: 0;
              width: 100%;
              text-align: center;
            }
            >.bed_num{
              height:.7rem;
              width:1.7rem;
              background-color: #303030;
              color:#fff;
              line-height: .8rem;
              text-align: center;
              position: absolute;
              top: 0;
              right: 0;
              font-size: .5rem;
            }
            >.occupyTime{
              position: absolute;
              top: .2rem;
              left: .3rem;
              font-size: .5rem;
              color:#ccc;
            }
          }
        }
      }
    }
  }
  .primary{
    width: 80%;
    margin: 1rem auto;
    >button[disabled].disabled{
      background-color: #ddd;
      color: #BFBFBF;
      border-color: #E5E9EA;
    }
  }
}
</style>
