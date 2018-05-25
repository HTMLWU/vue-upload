<template>
	<div class="finishOrderPage">
		<header class="wbHeader">
			<!-- <router-link to="/orderList/3" class="backPage"> 
				<i class="iconfont icon-left-angle"></i>
			</router-link> -->
			<a class="backPage" @click="backPage"><i class="iconfont icon-left-angle"></i></a>
			<span>完成维修</span>
		</header>
		<div class="finishOrderPageMain">
			<p class="getOrderPs">整改备注：</p>
			<textarea type="text" v-model="finishText" placeholder="请输入备注内容" class="finishText" maxlength="50"></textarea>
			<p class="getOrderPs">整改图片：</p>
			<div class="orderUploadImg">
				<div class="upload_warp_img" v-show="imgList.length!=0">
			        <div class="upload_warp_img_div" v-for="(item,index) in imgList">
			          <div class="upload_warp_img_div_top">
			            <div class="upload_warp_img_div_text">
			              {{item.file.name}}
			            </div>
			            <a  @click="fileDel(index)" class="upload_warp_img_div_del"><i class="iconfont icon-turnoff"></i></a>
			          </div>
			          <img :src="item.file.src">
			        </div>
			     </div>
				<a @click="fileClick" v-show="imgList.length<4"><i class="iconfont icon-Shape_fuzhi"></i></a>
			</div>
			 <input @change="fileChange($event)" type="file" id="upload_file" multiple style="display: none" >
		</div>
		<a class="finishOrderSubmit" @click="finishSubmit">提交</a>
	</div>
</template>
<script type="text/javascript">
	export default {
		data(){
			return {
				finishText:'',
				imgList:[],
				imgListStr:[],
				size: 0,
			}
		},
		mounted(){

		},
		methods:{
			fileClick(){
    			document.getElementById('upload_file').click();
			},
			fileChange(el){
		        if (!el.target.files[0].size) return;
		        this.fileList(el.target.files,el.target.value);
		        // console.log(el.target);
		        // el.target.value = ''
		    },
		    fileList(files,path){
		      	if(files.length>4){
		      		alert('最多只能上传4张图片');
		      		return false;
		      	}
		      	// this.formData = new FormData();
		        for (let i = 0; i < files.length; i++) {
		         	this.fileAdd(files[i]);
		        }
		        this.fileRequest(files);
		    },
		    backPage(){
				window.history.back();
			},
		    fileAdd(file){
		        this.size = this.size + file.size;//总大小
		        let reader = new FileReader();
		        reader.vue = this;
		        reader.readAsDataURL(file);
		        reader.onload = function () {
		          file.src = this.result;
		          this.vue.imgList.push({
		            file
		          });
		        }
		    },
		    fileDel(index){
		        this.size = this.size - this.imgList[index].file.size;//总大小
		        this.imgList.splice(index, 1);
		        this.imgListStr.splice(index, 1);
				 console.log(this.imgList);
				 console.log(this.imgListStr);

		    },
		    createX(){
                var xmlDom = null;
                if (window.XMLHttpRequest)
                    xmlDom = new XMLHttpRequest();
                else if (window.ActiveXObject)
                    xmlDom = new ActiveXObject("Microsoft.XMLHTTP");
                return xmlDom;
            },
            upload(url, fileSelect, call) {
                    var files = fileSelect;
                    var formData = new FormData();
                        var fis = files;
                        var type = fis.type;

                        //enctype="multipart/form-data"
                        formData.append('file',fis);
                        var xhr = this.createX();
                        xhr.open('POST', url, true);
                        xhr.onload = function () {
                            if (xhr.status != 200) {
                                alert('上传失败');
                                console.log('An error occurred!');
                            }
                        };

                        xhr.onreadystatechange = function () {
                            if (xhr.readyState == 4 && xhr.status == 200) {
                                if (call) {
                                    call(xhr);
                                }
                            }
                        }
                        xhr.send(formData);
            },
            fileRequest(el){
            	var uri = '/platform/socialunit/common/upload.do';
            	for (let i = 0; i < el.length; i++) {
            		let file= el[i];
	                this.upload(uri, file,(res)=>{
	                	this.getBackImg(res);
	                });
	            }   
            },
            getBackImg(res){
					let result = JSON.parse(res.responseText);
					let fileUrlArr = result.result.fileUrlArr;
					fileUrlArr.forEach(item=>{
						this.imgListStr.push(item);
					})
					 console.log(this.imgListStr);
            },
		    uploadImg(){
		    	/*var fdata = {
		    		filepath:this.formData
		    	}*/
		      	this.$http({
                    url: '/platform/socialunit/common/upload.do',
                    method: 'post',
                    params: this.formData
                }).then(succ => {
                  	
                }, error => {
               
                });
		    },
		    backPage(){
				window.history.back();
			},
			finishSubmit(){
				// var imglistString = JSON.stringify(this.imgListStr);
				var imglistString = this.imgListStr.join(",");
				this.$http({
                    url: '/platform/socialunit/workOrder/completeWorkOrder.do',
                    method: 'GET',
                    params: {
						workOrderId: this.$route.params.id,
                    	completeContent: this.finishText,
                       	completePic: imglistString
                    }
                }).then(succ => {
                	if(succ.data.code==0){
                  		this.backPage();
                	}else if(succ.data.code==1000){
                   		this.$toast(succ.data.message);
                	}
                }, error => {
               
                });
			},
		}
	}
</script>