<!--局部样式-->
<style scoped>
    .viewGraph3d {
        position: fixed;
        top: 60px;
        left: 0px;
        width: 100%;
        bottom: 0px;
    }
</style>

<!--局部覆盖样式-->
<style>
    .qqWindow {
        display: inline-flex;
        align-items: center;
        box-sizing: border-box;
        padding: 10px;
        background-color: rgba(51, 51, 51, 0.7);
        border-radius: 4px;
        font-size: 12px;
        color: #ddd;
    }

    .qqWindow > img {
        width: 100px;
        height: 100px;
        border: solid 1px #999;
    }

    .qqWindow > .infoWarp {
        width: 150px;
        height: 100px;
        margin-left: 10px;
    }

    .qqWindow > .infoWarp label {
        color: white;
        font-weight: bold;
    }

    .qqWindow > .infoWarp > div {
        height: 20px;
    }

    .qqWindow > .infoWarp > ul {
        box-sizing: border-box;
        list-style-type: none;
        margin: 0px;
        padding: 0px;
        height: 60px;
        overflow-y: auto;
        border: solid 1px #999;
    }
</style>

<template>
    <div class="viewGraph3d">
        <!-- <div class="qqWindow">
            <img :src="qqImgUrl('1982775886')" />
            <div class="infoWarp">
                <div>
                    <label>QQ:</label>
                    <span>1982775886</span>
                </div>
                <div>
                    <label>群内昵称：</label>
                </div>
                <ul>
                    <li>你好，世界</li>
                    <li>你好，世界</li>
                    <li>你好，世界</li>
                    <li>你好，世界</li>
                    <li>你好，世界</li>
                    <li>你好，世界</li>
                    <li>你好，世界</li>
                </ul>
            </div>
        </div> -->
        <div class="graph"></div>
    </div>
</template>

<script>
    import api from "@/api";
    import * as THREE from "three";
    import ForceGraph3D from "3d-force-graph";
    import SpriteText from "three-spritetext";

    export default {
        name: "viewGraph3d",
        props: {

        },
        data () {
            return {
                //#region 页面对象
                    //力导向图对象
                    graph: null,
                    //图片Map
                    imgMap: null,
                //#endregion

                //#region 页面内容绑定数据
                    searchType: "qq",
                    searchNum: "",
                //#endregion

                //#region 页面样式绑定数据
                //#endregion
            };
        },
        watch: {
            "$route": "handleRouteChange",
        },
        computed: {
            //#region 常量计算属性
            //#endregion

            //#region 数据转换计算属性
            //#endregion
            
            //#region 样式计算属性
            //#endregion
        },
        methods: {
            //#region 页面事件方法
                //路由改变触发更新事件
                handleRouteChange (nv) {
                    this.searchType = nv.query.search;
                    if (!this.searchType) {
                        this.searchType = "qq";
                    }
                    this.searchNum = nv.query.num;
                    if (!this.searchNum) {
                        this.searchNum = "10001";
                    }
                    this.b_updateGraph();
                },
            //#endregion

            //#region 业务逻辑方法
                //更新图表
                async b_updateGraph () {
                    let result = null;
                    if (this.searchType == "qq") {
                        result = await api.queryQQExtGraph(this.searchNum);
                    }
                    else if (this.searchType == "group") {
                        result = await api.queryGroupGraph(this.searchNum);
                    }
                    if (result) {
                        let data = this.graphData(result);
                        let keyList = this.imgKeyList(result);
                        await this.b_updateImgMap(result);
                        this.graph
                        .nodeId("nodeId")
                        .linkSource("sourceId")
                        .linkTarget("targetId")
                        .nodeAutoColorBy("nodeId")
                        .nodeThreeObject(node => {
                            // const sprite = new SpriteText(node.nodeLabel);
                            // sprite.color = node.color;
                            // sprite.textHeight = 8;
                            // return sprite;
                            return this.headBall(node);
                        })
                        .nodeLabel(node => {
                            let imgUrl = "";
                            if (node.nodeType == "group") {
                                imgUrl = this.groupImgUrl(node.nodeId);
                            }
                            else if (node.nodeType == "member") {
                                imgUrl = this.qqImgUrl(node.nodeId);
                            }
                            return this.qqWindowHtml(node);
                        })
                        .graphData(data);
                    }
                },
                //更新头像Map
                async b_updateImgMap (data) {
                    let keyList = this.imgKeyList(data);
                    this.imgMap = await this.getImgMap(keyList);
                },
            //#endregion

            //#region 资源访问方法
                //传入图片Url获取图片资源，返回一个Promise对象
                getImg (url, timeout = 60000) {
                    return new Promise((resolve, reject) => {
                        try {
                            let imgObj = new Image();
                            imgObj.src = url;
                            let id = setTimeout(() => {
                                reject();
                            }, timeout);
                            imgObj.onload = () => {
                                clearTimeout(id);
                                resolve(imgObj);
                            };
                            imgObj.onerror = () => {
                                reject();
                            };
                        }
                        catch (e) {
                            reject();
                        }
                    });
                },
                //新建一个用于绘制图像的Canvas
                createCanvas () {
                    let canvas = document.createElement("canvas");
                    canvas.width = 200;
                    canvas.height = 100;
                    let ctx = canvas.getContext("2d");
                    ctx.fillStyle = "#ffffff";
                    ctx.fillRect(0, 0, canvas.width, canvas.height);
                    return canvas;                
                },
                //把图像按照合适的方式绘制到空白Canvas之上
                drawImgToCanvas (img, cvs) {
                    let ctx = cvs.getContext("2d");
                    ctx.drawImage(img, 0, 0, 100, 100);
                    ctx.drawImage(img, 100, 0, 100, 100);
                },
                //根据图像生成适用于贴图的Canvas
                buildCanvas (img) {
                    let cvs = this.createCanvas();
                    this.drawImgToCanvas(img, cvs);
                    return cvs;
                },
            //#endregion

            //#region 数据转换方法
                //获取QQ头像的Url地址
                qqImgUrl (qqNum) {
                    return `/qqimg?dst_uin=${ qqNum }&spec=100`;
                },
                //获取群头像的Url地址
                groupImgUrl (groupNum) {
                    return `/groupimg/${ groupNum }/${ groupNum }/100`;
                },

                //根据接口返回数据格式整理出力导向图实际可用的数据格式
                graphData (data) {
                    let linkList = data.link.map(link => {
                        return {
                            sourceId: link.linkQQNum,
                            targetId: link.linkGroupNum,
                            linkLabel: link.linkNick,
                            linkValue: link,
                        };
                    });
                    let groupList = data.group.map(group => {
                        return {
                            nodeId: group.groupNum,
                            nodeType: "group",
                            nodeLabel: group.groupTitle,
                            nodeValue: group,
                        };
                    });
                    let memberList = data.member.map(member => {
                        return {
                            nodeId: member.memberQQNum,
                            nodeType: "member",
                            nodeLabel: "",
                            nodeValue: member,
                        };
                    });
                    //回查群成员节点群昵称信息
                    memberList.forEach(member => {
                        let link = linkList.find(link => link.sourceId == member.nodeId);
                        if (link) {
                            member.nodeLabel = link.linkLabel;
                        }
                        else {
                            member.nodeLabel = "未知QQ昵称";
                        }
                    });
                    let nodeList = groupList.concat(memberList);
                    return {
                        nodes: nodeList,
                        links: linkList,
                    };
                },

                //根据接口返回数据整理出去重后的图像Map键值
                imgKeyList (data) {
                    let list = data.group.map(item => `g${ item.groupNum }`);
                    list = list.concat(data.member.map(item => `m${ item.memberQQNum }`));
                    let dstSet = new Set(list);
                    return Array.from(dstSet);
                },

                //异步获取头像Map，这里返回一个Promise
                getImgMap (keyList) {
                    let imgMap = new Map();
                    return new Promise((resolve, reject) => {
                        try {
                            keyList.forEach(async keyStr => {
                                let imgUrl = "";
                                let num = keyStr.substring(1);
                                if (keyStr.startsWith("g")) {
                                    imgUrl = this.groupImgUrl(num);
                                }
                                else if (keyStr.startsWith("m")) {
                                    imgUrl = this.qqImgUrl(num);
                                }
                                try {
                                    let img = await this.getImg(imgUrl);
                                    imgMap.set(keyStr, img);
                                }
                                catch (e) {
                                    imgMap.set(keyStr, null);
                                }
                                if (imgMap.size == keyList.length) {
                                    resolve(imgMap);
                                }
                            });
                        }
                        catch (e) {
                            reject(e);
                        }
                    });
                },

                //生成成员信息浮动窗体
                qqWindowHtml (node) {
                    let temp = `
                        <div class="qqWindow">
                            <img src="${ this.qqImgUrl(node.nodeId) }" />
                            <div class="infoWarp">
                                <div>
                                    <label>QQ:</label>
                                    <span>${ node.nodeId }</span>
                                </div>
                                <div>
                                    <label>群内昵称：</label>
                                </div>
                                <ul>
                                    <li>${ node.nodeLabel }</li>
                                </ul>
                            </div>
                        </div>
                    `;
                    return temp;
                },

                headBall (node) {
                    let num = node.nodeId;
                    let type = node.nodeType;
                    let keyStr = "";
                    if (type == "group") {
                        keyStr = `g${ num }`;
                    }
                    else if (type == "member") {
                        keyStr = `m${ num }`;
                    }
                    let img;
                    if (this.imgMap.has(keyStr)) {
                        img = this.imgMap.get(keyStr);
                    }
                    else {
                        img = null;
                    }
                    let cvs = this.createCanvas();
                    this.drawImgToCanvas(img, cvs);
                    let texture = new THREE.Texture(cvs);
                    //新建标准网孔材质
                    let ballMat = new THREE.MeshStandardMaterial( {
                        color: "white",
                        roughness: 0.4,
                        metalness: 0.4,
                        map: texture,
                    });
                    texture.needsUpdate = true;
                    let ballGeometry = new THREE.SphereGeometry(3, 32, 32);
                    let ballMesh = new THREE.Mesh(ballGeometry, ballMat);
                    ballMesh.rotation.y = Math.PI;
                    return ballMesh;
                },
            //#endregion

            //#region 自动样式方法
            //#endregion

            //#region 其他方法
            //#endregion
        },
        created () {
            this.searchType = this.$route.query.search;
            if (!this.searchType) {
                this.searchType = "qq";
            }
            this.searchNum = this.$route.query.num;
            if (!this.searchNum) {
                this.searchNum = "10001";
            }
            this.b_updateGraph();
        },
        mounted () {
            const forceGraph3D = ForceGraph3D();
            this.graph = forceGraph3D(this.$el.querySelector(".graph"));
        },
        components: {

        },
    };
</script>