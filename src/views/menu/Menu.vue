<template>
    <div class="vue-template" >
        <div class="vue-template" >
            <div class="container-fluid" >
                <div class="row" >
                    <div class="col-sm-12" >
                        <div class="my-div" >
                            <div class="my-div-head" >
                                <div class="my-div-head-left" >
                                    <h3>Create menu</h3>
                                </div>
                                <div class="my-div-head-right" >
                                    <i class="fas fa-info-circle" ></i>
                                </div>
                            </div>
                            <div class="my-div-body" style="justify-content: flex-start" >
                                <div class="my-div-body-30" >

                                    <div class="menu-container">

                                        <div v-if="menu" class="my-menu" style="border-right: 1px solid #4CAF50;" >

                                            <p v-bind:class="{activeMenuStyle : selectedNode.i === 0  && selectedNode.j===-1 && selectedNode.k===-1}" v-on:click="selectedMenu(0,-1,-1)" >{{menu.text}}</p>

                                            <div class="menu-container"  >

                                                <div v-for="(m,j) in menu.children" class="my-menu" >

                                                    <p v-bind:class="{activeMenuStyle : selectedNode.i === 0  && selectedNode.j===j && selectedNode.k===-1}" v-on:click="selectedMenu(0,j,-1)" ><i v-bind:class="m.icon" ></i> {{m.text}}</p>

                                                    <div class="menu-container" >

                                                        <div  v-for="(s,k) in m.children" class="my-menu" >

                                                            <p v-bind:class="{activeMenuStyle : selectedNode.i === 0  && selectedNode.j===j && selectedNode.k===k}" v-on:click="selectedMenu(0,j,k)" >{{s.text}}</p>

                                                        </div>
                                                    </div>

                                                </div>
                                            </div>

                                        </div>
                                    </div>
                                </div>
                                <div class="my-div-body-20" >
                                    <table style="position: fixed" >
                                        <tbody>
                                        <tr>
                                            <td>
                                                <button class="my-btn" v-on:click="deleteChild" ><i class="fas fa-trash" ></i></button>
                                            </td>
                                        </tr>
                                        <tr>
                                            <td>
                                                <button class="my-btn" v-on:click="addChild" ><i class="fas fa-plus" ></i></button>
                                            </td>
                                        </tr>
                                        </tbody>
                                    </table>
                                </div>
                                <div class="my-div-body-50" >
                                    <table v-model="selectedMenuModel" >
                                        <tbody>
                                        <tr>
                                            <td>
                                                Text
                                            </td>
                                            <td>
                                                <input type="text" v-model="selectedMenuModel.text" />
                                            </td>
                                        </tr>
                                        <tr>
                                            <td>
                                                Icon
                                            </td>
                                            <td>
                                                <input type="text" v-model="selectedMenuModel.icon" />
                                            </td>
                                        </tr>
                                        <tr>
                                            <td>
                                                Link
                                            </td>
                                            <td>
                                                <input type="text" v-model="selectedMenuModel.link" />
                                            </td>
                                        </tr>
                                        </tbody>
                                    </table>
                                </div>
                            </div>
                            <div class="my-div-foot" >
                                <div class="my-div-foot-left" >
                                    <button class="my-btn" v-on:click="save" >Save</button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <notification ref="noti" ></notification>
    </div>
</template>

<script>

    import Notification from "../notificaiton/Notification";
    import CookieManager from "../../Helper/CookieManager";

    export default {
        name: "Menu",
        components: {Notification},
        mounted(){
          this.getInitialData();
        },
        data(){
            return{
                url : this.$store.state.baseUrl,
                menu : '',
                selectedNode : {
                    i : '',
                    j : '',
                    k : ''
                },
                isEditableFiledOpen : false,
                selectedMenuModel : {
                    link : '',
                    text: '',
                    icon: '',
                },
                needToCloseNotification : true
            }
        },
        methods : {
            getInitialData(){

                this.$refs.noti.setNotificationProperty({
                    title : 'Loading',
                    bodyIcon : 'fas fa-sync fa-spin',
                    bodyMsg : 'Please wait ... !'
                });

                this.$http.post(this.url+"/menu/get-by-department",{
                    departmentBn : {
                        id : CookieManager.getParsedData("userInfo").deptId
                    },
                    userBn : CookieManager.getParsedData("userInfo"),
                    menuBn : {
                        link: this.$route.path
                    }
                })
                .then(res=>{

                    console.log(JSON.stringify(res.data));

                    if (res.status===200){

                        this.menu = res.data.menuBn;
                        if (this.needToCloseNotification){
                            this.$refs.noti.closeNotification();
                        }

                    } else {
                        this.$refs.noti.setNotificationProperty({
                            title : 'Error',
                            bodyIcon : 'fas fa-exclamation-circle',
                            bodyMsg : 'Can not get menu !',
                            callBackMethod : this.getInitialData,
                            needTryAgain : true,
                            status : 400
                        });
                    }

                })
                .catch(err=>{
                    console.log(err);
                    this.$refs.noti.setNotificationProperty({
                        title : 'Error',
                        bodyIcon : 'fas fa-exclamation-circle',
                        bodyMsg : 'Can not get menu !',
                        callBackMethod : this.getInitialData,
                        needTryAgain : true,
                        status : 400
                    });
                })
            },
            selectedMenu(i,j,k){

                this.selectedNode.i = i;
                this.selectedNode.j = j;
                this.selectedNode.k = k;

                if (this.selectedNode.i !== -1 && this.selectedNode.j === -1 && this.selectedNode.k === -1) {

                    this.selectedMenuModel = this.menu;

                }else if (this.selectedNode.i !== -1 && this.selectedNode.j !== -1 && this.selectedNode.k === -1){

                    this.selectedMenuModel = this.menu.children[this.selectedNode.j];

                }else {

                    this.selectedMenuModel = this.menu.children[this.selectedNode.j].children[this.selectedNode.k];

                }

            },
            up(){

                try {

                    if (this.selectedNode.i !== -1 && this.selectedNode.j === -1 && this.selectedNode.k === -1) {

                        let x = this.menu[this.selectedNode.i].srl;
                        let y = this.menu[this.selectedNode.i-1].srl;

                        this.menu[this.selectedNode.i].srl = y;
                        this.menu[this.selectedNode.i-1].srl = x;

                        this.menu.sort(function(a, b){
                            return a.srl - b.srl;
                        });

                    }else if (this.selectedNode.i !== -1 && this.selectedNode.j !== -1 && this.selectedNode.k === -1){

                        if (this.selectedNode.i === 0 && this.selectedNode.j === 0 && this.selectedNode.k === -1){
                            console.log("last");
                        }else {

                            let x = this.menu[this.selectedNode.i].children[this.selectedNode.j].srl;
                            let y = this.menu[this.selectedNode.i].children[this.selectedNode.j-1].srl;

                            this.menu[this.selectedNode.i].children[this.selectedNode.j-1].srl = x;
                            this.menu[this.selectedNode.i].children[this.selectedNode.j].srl = y;

                            this.selectedNode.j = this.selectedNode.j-1;

                            this.menu[this.selectedNode.i].children.sort(function(a, b){
                                return a.srl - b.srl;
                            });

                        }

                    } else {

                        if (this.selectedNode.k > 0){

                            let x = this.menu[this.selectedNode.i].children[this.selectedNode.j].children[this.selectedNode.k].srl;
                            let y = this.menu[this.selectedNode.i].children[this.selectedNode.j].children[this.selectedNode.k-1].srl;

                            this.menu[this.selectedNode.i].children[this.selectedNode.j].children[this.selectedNode.k-1].srl = x;
                            this.menu[this.selectedNode.i].children[this.selectedNode.j].children[this.selectedNode.k].srl = y;

                            this.selectedNode.k = this.selectedNode.k-1;

                            this.menu[this.selectedNode.i].children[this.selectedNode.j].children.sort(function(a, b){
                                return a.srl - b.srl;
                            });

                        }else {

                            if (this.selectedNode.i===0 && this.selectedNode.j===0 && this.selectedNode.k===0){
                                console.log("last");
                            }else {
                                let obj = this.menu[this.selectedNode.i].children[this.selectedNode.j].children[this.selectedNode.k];
                                this.menu[this.selectedNode.i].children[this.selectedNode.j].children.splice(this.selectedNode.k,1);

                                let cLn = this.menu[this.selectedNode.i].children[this.selectedNode.j-1].children.length;

                                obj.srl = cLn+1;

                                this.menu[this.selectedNode.i].children[this.selectedNode.j-1].children.push(obj);

                                this.menu[this.selectedNode.i].children[this.selectedNode.j-1].children.sort(function(a, b){
                                    return a.srl - b.srl;
                                });

                                this.selectedNode.k = cLn;
                                this.selectedNode.j = this.selectedNode.j-1;
                            }

                        }

                    }

                }catch (e) {

                }

            },
            down(){

                if (this.selectedNode.i !== -1 && this.selectedNode.j === -1 && this.selectedNode.k === -1) {

                    let x = this.menu[this.selectedNode.i].srl;
                    let y = this.menu[this.selectedNode.i+1].srl;

                    this.menu[this.selectedNode.i].srl = y;
                    this.menu[this.selectedNode.i+1].srl = x;

                    this.menu.sort(function(a, b){
                        return a.srl - b.srl;
                    });

                }else if (this.selectedNode.i !== -1 && this.selectedNode.j !== -1 && this.selectedNode.k === -1){

                    let x = this.menu[this.selectedNode.i].children[this.selectedNode.j].srl;
                    let y = this.menu[this.selectedNode.i].children[this.selectedNode.j+1].srl;

                    this.menu[this.selectedNode.i].children[this.selectedNode.j+1].srl = x;
                    this.menu[this.selectedNode.i].children[this.selectedNode.j].srl = y;

                    this.selectedNode.j = this.selectedNode.j+1;

                    this.menu[this.selectedNode.i].children.sort(function(a, b){
                        return a.srl - b.srl;
                    });

                } else {

                    let ln = this.menu[this.selectedNode.i].children[this.selectedNode.j].children.length;
                    console.log(this.selectedNode.i+"|"+this.selectedNode.j+"|"+this.selectedNode.k+"| ln "+ln);
                    if (ln===this.selectedNode.j){
                        console.log("hi");
                    }

                    if (this.selectedNode.k !== (ln-1)){

                        let x = this.menu[this.selectedNode.i].children[this.selectedNode.j].children[this.selectedNode.k].srl;
                        let y = this.menu[this.selectedNode.i].children[this.selectedNode.j].children[this.selectedNode.k+1].srl;

                        this.menu[this.selectedNode.i].children[this.selectedNode.j].children[this.selectedNode.k+1].srl = x;
                        this.menu[this.selectedNode.i].children[this.selectedNode.j].children[this.selectedNode.k].srl = y;

                        this.selectedNode.k = this.selectedNode.k+1;

                        this.menu[this.selectedNode.i].children[this.selectedNode.j].children.sort(function(a, b){
                            return a.srl - b.srl;
                        });

                    }else {


                        console.log("c"+this.menu[this.selectedNode.i].children[this.selectedNode.j].children.indexOf(this.selectedNode.k));

                        let obj = this.menu[this.selectedNode.i].children[this.selectedNode.j].children[this.selectedNode.k];
                        this.menu[this.selectedNode.i].children[this.selectedNode.j].children.splice(this.selectedNode.k,1);

                        obj.srl = 1;

                        this.menu[this.selectedNode.i].children[this.selectedNode.j+1].children.push(obj);

                        for (let i = 0; i < this.menu[this.selectedNode.i].children[this.selectedNode.j+1].children.length-1; i++) {
                            this.menu[this.selectedNode.i].children[this.selectedNode.j+1].children[i].srl = i+2;
                        }

                        this.menu[this.selectedNode.i].children[this.selectedNode.j+1].children.sort(function(a, b){
                            return a.srl - b.srl;
                        });

                        this.selectedNode.k = 0;
                        this.selectedNode.j = this.selectedNode.j+1;

                    }

                }

            },
            save(){

                console.log(JSON.stringify(this.menu));

                this.$refs.noti.setNotificationProperty({
                    title : 'Processing',
                    bodyIcon : 'fas fa-sync fa-spin',
                    bodyMsg : 'Please wait !',
                });

                this.$http.post(this.$store.state.baseUrl+"/menu/save",{
                    userBn : CookieManager.getParsedData("userInfo"),
                    menuBn : {
                        id : this.menu.id,
                        oId : this.menu.oId,
                        text : this.menu.text,
                        parentId : this.menu.parentId,
                        rk : this.menu.rk,
                        srl : this.menu.srl,
                        link : this.$route.path,
                        modifiedBy : this.menu.modifiedBy,
                        children : this.menu.children
                    }

                }).then(res=>{

                    console.log(JSON.stringify(res.data));

                    if (res.data.code===200){

                        this.needToCloseNotification = false;

                        this.$refs.noti.setNotificationProperty({
                            title : 'Success',
                            bodyIcon : 'fas fa-check-circle',
                            bodyMsg : res.data.msg,
                            code: res.data.code
                        });

                    }else {
                        this.$refs.noti.setNotificationProperty({
                            title : 'Error',
                            bodyIcon : 'fas fa-exclamation-circle',
                            bodyMsg : res.data.msg,
                            callBackMethod : this.getInitialData,
                            needTryAgain : true,
                            code : res.data.code
                        });
                    }

                }).catch(err=>{
                    console.log(JSON.stringify(err.response.data));
                    this.$refs.noti.setNotificationProperty({
                        title : 'Error',
                        bodyIcon : 'fas fa-exclamation-circle',
                        bodyMsg : err.response.data.message,
                        callBackMethod : this.save,
                        needTryAgain : true,
                        code : err.response.data.status
                    });
                })
            },
            addChild(){
                if (this.selectedNode.i !== -1 && this.selectedNode.j === -1 && this.selectedNode.k === -1) {
                    // root
                    this.menu.children.push({
                        id: 0,
                        oId: 0,
                        text: 'New child',
                        icon: '',
                        parentId: 0,
                        rk: 0,
                        srl: 0,
                        children: []
                    });
                }else if (this.selectedNode.i !== -1 && this.selectedNode.j !== -1 && this.selectedNode.k === -1){
                    // menu
                    this.menu.children[this.selectedNode.j].children.push({
                        id: 0,
                        oId: 0,
                        text: 'New child',
                        icon: '',
                        parentId: 0,
                        rk: 0,
                        srl: 0,
                        menuPermissionList: []
                    });
                }else {
                    // child
                    this.$refs.noti.setNotificationProperty({
                        title : 'Error',
                        bodyMsg : 'Sorry, Sir/Mam you can not add child of the menu'
                    });
                }
            },
            deleteChild(){

                if (this.selectedNode.i !== -1 && this.selectedNode.j === -1 && this.selectedNode.k === -1) {
                    // root

                    if (this.menu.children.length>0){
                        this.$refs.noti.setNotificationProperty({
                            title : 'Error',
                            bodyMsg : 'Sorry, Sir/Mam you can not delete this menu cause this menu got child'
                        });
                    }

                }else if (this.selectedNode.i !== -1 && this.selectedNode.j !== -1 && this.selectedNode.k === -1){
                    // menu

                    if (this.menu.children[this.selectedNode.j].children.length>0){
                        this.$refs.noti.setNotificationProperty({
                            title : 'Error',
                            bodyMsg : 'Sorry, Sir/Mam you can not delete this menu cause this menu got child'
                        });
                    } else {
                        this.menu.children.splice(this.selectedNode.j,1);
                    }

                }else {

                    if (this.menu.children[this.selectedNode.j].children[this.selectedNode.k].menuPermissionBnList.length===0){

                        this.menu.children[this.selectedNode.j].children.splice(this.selectedNode.k,1);

                    } else {
                        this.$refs.noti.setNotificationProperty({
                            title : 'Error',
                            bodyMsg : 'Sorry, Sir/Mam you can not delete this menu cause this menu is permitted with some department !'
                        });
                    }

                }
            }
        }
    }
</script>

<style scoped>
    .menu-container{
        margin-left: 40px;
    }
    .my-menu{
        border-left: 1px solid #4CAF50;
    }
    .my-menu p{
        font-size: 15px;
        border-top: 1px solid lightgray;
        border-bottom: 1px solid lightgray;
        border-left: 1px solid lightgray;
        padding: 3px;
        margin: 2px;
    }
    .my-menu p:hover{
        cursor: grab;
    }
    .activeMenuStyle{
        background-color: #4CAF50;
        color: white;
        border-radius: 3px;
    }
</style>
