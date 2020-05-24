<template>
    <div>
        <el-upload
          class="upload-demo"
          action
          accept='.xlsx,.xls'
          :on-change='handle'
          :auto-upload='false'
          >
          <el-button size="small" type="primary">点击上传</el-button>
        </el-upload>
    </div>
</template>

<script>
  import xlsx from 'xlsx'
    export default {
      data(){
        return {
          character: {
            name: {
              text: '姓名',
              type: 'string'
            },
            phone: {
              text: '号码',
              type: 'string'
            }
          }
        }
      },
        methods: {
         async handle(ev){
            let file = ev.raw
            if(!file) return
            let data = await this.readFile(file)
            let workbook = xlsx.read(data, {type: 'binary'}),
                worksheet = workbook.Sheets[workbook.SheetNames];
                data = xlsx.utils.sheet_to_json(worksheet)
                let arr = []
              data.forEach(item=> {
                let obj = {}
                for(let key in this.character){
                  if(!this.character.hasOwnProperty(key))break;
                  let value = this.character[key],
                  text = value.text,
                  type = value.type;
                  value = item[text];
                  value === 'string' ? String(value) : null;
                  obj[key] = value
                }
                arr.push(obj)
              })  
              console.log(arr)
          },
          readFile(file){
            return new Promise(resolve=>{
              let reader = new FileReader();
              reader.readAsBinaryString(file)//将文件转化为二进制
              reader.onload = ev=>{
                resolve(ev.target.result)
              }
            })
          }
        }
    }
</script>

<style lang="scss" scoped>

</style>