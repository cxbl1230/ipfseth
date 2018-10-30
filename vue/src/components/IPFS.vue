<template>
  <div class="container">
   <el-upload 
   class="upload-demo" 
   action="" 
   ref="upload" 
   :auto-upload="true" 
   :before-upload="onBefore"
   accept="image/jpeg,image/gif,image/png,image/bmp"
   multiple>
   <el-button size="small" type="primary">点击上传</el-button>
  </el-upload>
  <br />
    <el-button size="small" type="success" @click="getEth">获取</el-button>

  </div>
</template>

<script>
const ipfsAPI = require('ipfs-api')
let contractInstance;


export default {
  name: 'IPFS',
  data () {
    return {
        abi:[{"constant": false,"inputs": [{"name": "x","type": "string"}],"name": "set","outputs": [],"payable": false,"stateMutability": "nonpayable","type": "function"},{"constant": true,"inputs": [],"name": "get","outputs": [{"name": "x","type": "string"}],"payable": false,"stateMutability": "view","type": "function"}],
        address:'0x76d67ce466789c471787732325eee15b47a56d00',
        from:'0xD6084bC70Ee9267E7E42E8A7ddB4e2E59c17D42D'
    }
  },
  methods:{
    upload2ipfs(render){
        let that = this
        // connect to ipfs daemon API server
        var ipfs = ipfsAPI('localhost', '5001', {protocol: 'http'})
        let buffer = Buffer.from(render.result);
        ipfs.add(buffer).then((response) => {
            that.sendEth(response[0].hash)
            //resolve(response[0].hash);
        }).catch((err) => {
            console.error(err)
            //reject(err);
        })
    },
    onBefore(file){
        let that = this
        let filename = window.URL.createObjectURL(file)
        let render = new FileReader()
        render.readAsDataURL(file)
        render.onload = function () {
            that.upload2ipfs(render)
        }
        return false
    },
    sendEth(string){
        let that = this
        contractInstance.methods.set(string).send({from: that.from})
        .then(function(response){
            console.log(response)
        });
    },
    getEth(){
        contractInstance.methods.get().call().then(function(res){
            console.log(res)
        });
    }
  },
    mounted () {
        let that = this
        contractInstance = new this.$web3.eth.Contract(that.abi,that.address)
    }
}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
