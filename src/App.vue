<script setup>
import { ref, onMounted, computed } from "vue";
import Web3 from "web3";
import mtcJSON from "./contract/MyToken.json";

const goerLiWS =
  "wss://goerli.infura.io/ws/v3/dc92288a859d445890ede39d9dba5861";

const web3 = new Web3(Web3.givenProvider || goerLiWS);

//0xd9145CCE52D386f254917e481eB44e9943F39138
const metContract = new web3.eth.Contract(
  mtcJSON.abi
  //,"0xd9145CCE52D386f254917e481eB44e9943F39138"
);

const name = ref("");
const symbol = ref("");
const totalSupply = ref("");
const balanceOf = ref(0);
const currentAccount = ref("");

//转账信息
const toAddress = ref("")
const amount = ref(0)


const getCoinInfo = async () => {
  const account = await web3.eth.requestAccounts();
  currentAccount.value = account[0];
  name.value = await metContract.methods.name().call();
  symbol.value = await metContract.methods.symbol().call();
  totalSupply.value = await metContract.methods.totalSupply().call();
  balanceOf.value = await metContract.methods.balanceOf(account[0]).call();
};

const ts = computed(()=>{
  return web3.utils.fromWei(totalSupply.value,'ether')
})

onMounted(() => {
  getCoinInfo();
});

const send = ()=>{
  const weiAomunt = web3.utils.toWei(amount.value,'ether');

  metContract.methods.transfer(toAddress,weiAomunt).send({
    from:currentAccount.value[0],
  })
  .on("receipt",(res)=>{
    console.log("交易成功！");
    console.log(res);
  })
}

// ( async()=>{
//   const account = await web3.eth.requestAccounts();
//   console.log("account--->",account)
// })();
</script>

<template>
  <h1>基础信息</h1>
  <hr />
  <p>当前账号：{{ currentAccount }}</p>
  <p>代币名称：{{ name }}</p>
  <p>代币标识：{{ symbol }}</p>
  <p>发行量：{{ totalSupply }}</p>
  <p>持有数量：{{ balanceOf }}</p>
  <h1>转装操作</h1>
<hr/>
<p>转入地址：<input type="text" v-model="toAddress"/></p>
<p>转出金额：<input type="text" v-model="amount"/></p>
<button @click="send">开始转账</button>
</template>

<style lang="less">
</style>