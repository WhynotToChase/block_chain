<template>
  <div id="a" v-loading="loading" :element-loading-text="loadingText">
    <div v-if="show" id='e'>
      <div id='b'>
        <div id='g'></div>
        <h2>当前拥有的专利</h2>
        <el-button type='primary' @click="refresh" id='g'>刷新</el-button>
      </div>

      <el-table :data="paginatedPatents" border id="c">
        <el-table-column fixed prop="patentID" label="专利ID" width="150" />
        <el-table-column prop="patentDetail" label="专利描述" width="1200" show-overflow-tooltip />
        <el-table-column fixed="right" label="操作" width="100">
          <template #default="{row}">
            <el-button type="primary" size="small" @click="onsale(row)">出售</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页组件 -->
      <div id="d">
        <el-pagination
            background
            layout="prev, pager, next, jumper"
            :total="patents.length"
            :page-size="pageSize"
            :current-page="currentPage"
            @current-change="handlePageChange"
            :pager-count="10"
        />
      </div>
    </div>
    <div v-else id="m">
      <div id='n'>
        <div id="h">
          <el-button type="primary" @click="back" id='j'>返回</el-button>
          <h2>售卖专利</h2>
          <div id ='j'></div>
        </div>

        <div id='i'>
          <el-card v-for="patent in patents" :key="patent.patentID" id='k'>
            <template #header>
              <div>
                <span>专利ID：{{patent.patentID}}</span>
              </div>
            </template>
            <div id="l">
              <el-scrollbar>
                <el-text>专利描述：<br>{{patent.patentDetail}}</el-text>
              </el-scrollbar>
            </div>
          </el-card>
        </div>
      </div>
      <el-divider direction="vertical">
        <el-icon><star-filled /></el-icon>
      </el-divider>
      <div id='o'>
        <div>
          <span>请输入价格：</span>
          <el-input-number v-model="price" :precision="2" :step="100" />
          <span>人民币/￥</span>
        </div>
        <div id='p'>
          <el-input show-password placeholder="请输入密码" v-model="password" clearable></el-input>
        </div>
        <el-button type="warning" @click="comfirmSale">出售</el-button>
      </div>
    </div>
  </div>
</template>
 
<script setup name="PatentSell" lang="ts">
import { ElMessage, ElMessageBox } from 'element-plus';
import {computed, onMounted, ref} from 'vue';
import {PatentMarket, web3} from "@/web3.ts";

interface Patent {
  patentID: number;
  patentDetail: string;
}

let patents = ref<Patent[]>([]);
let pageSize = ref(10); // 每页显示行数
let currentPage = ref(1); // 当前页码

// 计算当前页显示的数据
const paginatedPatents = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value;
  const end = start + pageSize.value;
  return patents.value.slice(start, end);
});
let loading=ref(false);
let onsalepatent = ref<Patent | null>(null);
function handlePageChange(page:number){
	currentPage.value=page;
}

async function getPatents(){
	loading.value = true;
	try {
		//获取用户名下的专利
    // 获取当前用户地址
    const accounts = await web3.eth.getAccounts();
    const patentHolder = accounts[0];
    console.log('patentHolder:', patentHolder);
    // Call the contract's fetchPatentsCanSale function
    const userPatentsForSale = await PatentMarket.methods.fetchPatentsCanSale(patentHolder).call();

    console.log('userPatentsForSale:', userPatentsForSale);

    // Ensure the response is an array before mapping
    if (Array.isArray(userPatentsForSale)) {
      patents.value = userPatentsForSale.map(patent => ({
        patentID: patent.patentId,
        patentDetail: patent.description
      }));
    } else {
      patents.value = [];
    }
	}
	catch(error){
		ElMessage({message:'获取专利信息失败',type:'warning'});
	}
	finally{ 
		loading.value=false; 		
	}
}
onMounted(()=>{
	loadingText.value="查询专利中";
	getPatents();
})
function refresh(){
	loadingText.value="查询专利中";
	getPatents();
}
let show=ref(true);

function onsale(row: Patent) {
  show.value = false;
  onsalepatent.value = row;
}

function back(){
	show.value=true;
}

let price=ref(0);
let password=ref("");
let loadingText=ref("")

async function comfirmSale() {
  loading.value = true; // 开始查询时显示加载状态
  loadingText.value = "正在交易";
  try {
    // 获取当前用户地址
    const accounts = await web3.eth.getAccounts();
    const seller = accounts[0];

    // 调用合约的 listForSale 函数
    await PatentMarket.methods.listForSale(onsalepatent.value.patentID, price.value).send({ from: seller });

    ElMessageBox.alert('专利已成功上架出售', '交易成功', {
      confirmButtonText: '确认',
      callback: () => {
        ElMessage({
          type: 'success',
          message: '已返回',
        });
        show.value = true; // 或者 router.back();
      },
    });
  } catch (error) {
    const errorMessage = error instanceof Error ? error.message : '未知错误';
    ElMessageBox.alert(errorMessage, '交易失败', {
      confirmButtonText: '确认',
      callback: () => {
        ElMessage({
          type: 'error',
          message: '已返回',
        });
        show.value = true; // 或者 router.back();
      },
    });
  } finally {
    loading.value = false;
  }
}
</script>

<style scoped>
#a {
  position: relative;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.084);
}

#a .el-loading-spinner {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

	
#b{
	height: 50px;
	width: 1000px;
	display: flex;
	justify-content: space-between;
}
#c{
	width: 1100px;
	height: 450px;
	margin:0 auto;
}
#e{
	
	margin-top: -50px;
	display: flex;
	flex-direction: column;
	justify-content:space-around;
	height:600px;
	width: 1300px;
	background-color: #ffffff;
	border-radius: 30px;
	align-items: center;
}
#g{
	width: 80px;
	height: auto;
}
#h{
	width: 500px;
	display: flex;
	justify-content: space-between;
	align-items: center;
	flex-direction: row;
	height: 50px;
}
#j{
	width: 80px;
}
#k{
	margin-top: 20px;
	width: 500px;
	height: 400px;
}
#m{
	margin-top: -50px;
	display: flex;
	justify-content:space-around;
	height:600px;
	width: 1300px;
	background-color: #ffffff;
	border-radius: 30px;
	align-items: center;
	
}
#n{
	display: flex;
	flex-direction: column;
	justify-content:center;
	height:600px;
	width: 500px;
	align-items: center;
}
#l{
	height: 350px;
}
#o{
	height: 200px;
	width: 500px;
	display: flex;
	flex-direction: column;
	justify-content: space-around;
	align-items: center;
}
#p{
	width: 350px;
}

</style>