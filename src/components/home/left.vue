<template>
    <div class="menu">
        <div class="column">
            <p>在线音乐<a ondragstart="return false" href="#"><i class="iconfont icon-tubiaozhizuomoban"></i>反馈</a></p>
            <ul>
                <li v-for="(item, index) in onlineMusic" :key="index">
                    <router-link ondragstart="return false" :to="item.action">
                        <i :class="'iconfont icon-' + item.icon"></i>
                        <span>{{item.txt}}</span>
                    </router-link>
                </li>
            </ul>
        </div>
        <div class="column">
            <p>我的音乐</p>
            <ul>
                <li v-for="(item, index) in myMusic" :key="index">
                    <router-link ondragstart="return false" :to="item.action">
                        <i :class="'iconfont icon-' + item.icon"></i>
                        <span>{{item.txt}}</span>
                    </router-link>
                </li>
            </ul>
        </div>
        <div class="column">
            <p>自建歌单
                <span>
                    <a ondragstart="return false" href="#" @click="showList = !showList"><i class="iconfont" :class="showList ? 'icon-Group-' : 'icon-shang-'"></i></a>
                    <a ondragstart="return false" href="#" @click="ShowInput"><i style="font-size:12px;" class="iconfont icon-jia"></i></a>
                </span>
            </p>
            <ul :class="showList ? 'hide' : 'show'">
                <!-- 右键有 播放歌单 下一首播放 分享 -->
                <li v-for="(item, index) in songList" @mousedown="ShowMenu($event, item.txt, index, 0, 'songList')" :key="index">
                    <router-link ondragstart="return false" :to="{
                        path: item.action,
                        query: item.query || ''
                    }">
                        <i :class="'iconfont icon-' + item.icon"></i>
                        <span>{{item.txt}}</span>
                    </router-link>
                </li>
                <!-- 右键有 播放歌单 下一首播放 重命名 删除歌单 分享 -->
                <li v-for="(item, i) in myLikeMusic" @mousedown="ShowMenu($event, item.txt, i, 1, 'colLikeMusic')" :key="i.a">
                    <router-link ondragstart="return false" :to="{
                        path: item.action,
                        query: item.query || ''
                    }">
                        <i :class="'iconfont icon-' + item.icon"></i>
                        <span>{{item.txt}}</span>
                    </router-link>
                </li>
                <li class="addList" v-show="showInput">
                    <i class="iconfont icon-ziyuan"></i>
                    <input type="text" ref="addInput" @focus="clearHotKey($event)" @blur="saveCol" @keydown.13="$event.target.blur" :value="'我的歌单' + addList" style="width: 100px; vertical-align: middle;">
                </li>
            </ul>
        </div>
        <div class="column">
            <p>收藏歌单
                <span>
                    <a ondragstart="return false" href="#" @click="showCollecting = !showCollecting"><i class="iconfont" :class="showCollecting ? 'icon-Group-' : 'icon-shang-'"></i></a>
                </span>
            </p>
            <ul :class="showCollecting ? 'hide' : 'show'">
                <!-- 右键有 播放歌单 下一首播放 删除歌单 分享 -->
                <li v-for="(item, index) in collectingSongs" :key="index" @mousedown="ShowMenu($event, item, index, 2, 'collectingSongs')">
                    <router-link ondragstart="return false" :to="comAction(item.action)" :class="{active: item.active}" @click.native="toggleActive();item.active = !item.active">
                        <i :class="'iconfont icon-' + item.icon"></i>
                        <span>{{item.txt}}</span>
                    </router-link>
                </li>
            </ul>
        </div>
        <div class="layer mouseMenu" v-show="showMenu" @click="showMenu = false">
            <div class="mMenu mouse">
                <a ondragstart="return false" href="#" @click="playMusic">播放歌单</a>
                <a ondragstart="return false" href="#">下一首播放</a>
                <a ondragstart="return false" href="#" :class="menuType === 1 ? 'show' : 'hide'" @click="showRename">重命名</a>
                <a ondragstart="return false" href="#" :class="menuType === 0 ? 'hide' : 'show'" @click="deleteCol">删除歌单</a>
                <a ondragstart="return false" href="#">分享</a>
            </div>
        </div>
        <!-- showRepeatName && menuType == 1 -->
        <div class="layer" v-show="showRepeatName" @click="showRepeatName = false">
            <div class="repeat-name" @click.stop="">
                <p><a ondragstart="return false" href="#" @click.stop="showRepeatName = false"><i class="iconfont icon-cha"></i></a><span>重命名</span></p>
                <div class="content">
                    <p class="header">标题:</p>
                    <input type="text" placeholder="请输入新的歌单名" ref="rename" @focus="clearHotKey($event)" @blur="addHotKey">
                    <p class="btn">
                        <a ondragstart="return false" href="#">取消</a>
                        <a ondragstart="return false" href="#" @click="rename" >确定</a>
                    </p>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            onlineMusic: [
                { txt: '推荐', icon: 'yinle', active: true, action: '/' },
                { txt: '电台', icon: 'xinhao', active: false, action: '/radiostation' },
                { txt: '歌单', icon: 'bofang', active: false, action: '/songsheet' },
                { txt: '排行榜', icon: 'paihangbang', active: false, action: '/rankingList' },
                { txt: '歌手', icon: 'geshou', active: false, action: '/singer' },
                { txt: 'MV', icon: 'shipinbofangyingpian', active: false, action: '/mv' }
            ],
            myMusic: [
                { txt: '下载管理', icon: 'xiazai', active: false, action: '/download' },
                { txt: 'iTunes', icon: 'yinle1', active: false, action: '/iTunes' }
            ],
            addList: 1, // 新建歌单默认的value 如: '我的歌单' + 1
            showInput: false, // 是否显示新建歌单的input
            showList: false, // 显示/隐藏 自建歌单
            showCollecting: false, // 显示/隐藏 收藏歌单
            showMenu: false, // 更多操作菜单
            menuType: 0, // 三种菜单类型 类似根据权限显示菜单
            musicIndex: null,
            item: null,
            showRepeatName: false,
            currentColName: ''
        }
    },
    methods: {
        // 处理路径
        comAction(src){
            let arr = new String(src).split('?')
            return {
                path: arr[0],
                query: {
                    id: arr[1]
                }
            }
        },
        clearHotKey(e) {
            // 输入框获取焦点 取消全局快捷键
            document.onkeydown = null

        },
        addHotKey(){
            // 输入框失去焦点 判断是否添加全局快捷键
            this.showInput = false
            if (this.$store.state.hotKey) {
                document.onkeydown = e => {
                    this.$store.commit('RegisterHotKey', e)
                }
            } else {
                document.onkeydown = null
            }

        },
        saveCol(){
            this.addHotKey()
            // 把输入好的歌单保存
            let value = this.$refs.addInput.value
            let flag = true
            for(let i = 0; i< this.myLikeMusic.length; i++){
                if(this.myLikeMusic[i].txt === value){
                    flag = false
                    break;
                }else{
                    flag = true
                }
            }

            if(flag){
                // 歌单名字兼容
                if(value == '我的歌单' + this.addList){
                    this.addList++
                }
                // 把新建歌单的名称以及路径 放进 自建歌单
                this.$store.state.myLikeMusic.push({
                    txt: value,
                    icon: 'ziyuan',
                    active: false,
                    action: '/mylike',
                    query: {
                        list: value
                    }
                })
                // 新建空的歌单歌曲列表
                localStorage.setItem(value,JSON.stringify([]))
            }else{
                alert('列表名称重复,请重新修改')
            }

        },
        toggleActive() {
            this.onlineMusic.forEach(item => {
                item.active = false
            })
            this.myMusic.forEach(item => {
                item.active = false
            })
            this.$store.state.songList.forEach(item => {
                item.active = false
            })
            this.collectingSongs.forEach(item => {
                item.active = false
            })
        },
        ShowInput(){
            this.showInput = true
            setTimeout(() => {
                this.$refs.addInput.focus()
            },100)
        },
        ShowMenu(e, item, index, menuType, colName){
            this.currentColName = colName
            // 判断是否是右键按下
            if (e.button === 2) {
                this.showMenu = true
                this.menuType = menuType

                this.musicIndex = index
                this.item = item

                var menu = document.getElementsByClassName('mouse')[0]
                var limit = document.getElementsByClassName('limit')[0]

                menu.style.left = e.clientX - this.getElementTop(limit, 'Left') + 'px'
                menu.style.top = e.clientY - this.getElementTop(limit, 'Top') + 'px'

            }
        },
        playMusic(){
            if(this.menuType !== 2){
                // 我喜欢 默认收藏 和自己新建的歌单
                this.$store.state.playerList =  JSON.parse(localStorage.getItem(this.item)) || [];
                this.$store.state.currentPlayerIndex = 0
                this.$store.commit('searchMusic')
                this.$store.commit('pushHistoryMusic')
            }else{
                // 收藏歌单
                this.requstSongsheet(this.item.id)
            }
        },
        // 删除歌单
        deleteCol(){
            if(confirm('此操作不可逆，是否继续？')){

                if(this.menuType === 1){
                    localStorage.removeItem(this.item)
                    let arr = JSON.parse(localStorage.getItem('myLikeMusic')) || []
                    this.$store.state.myLikeMusic = arr.filter((item) => item.txt !== this.item)
                }
                if(this.menuType === 2){
                    let arr = JSON.parse(localStorage.getItem('collectingSongs')) || []
                    this.$store.state.collectingSongs = arr.filter((item) => item.txt !== this.item.txt)
                }
            }
        },
        showRename(){
            this.$refs.rename.value = ''
            this.showRepeatName = true
            setTimeout(() => {
                this.$refs.rename.focus();
            }, 200)
        },
        // 重命名
        rename(){
            if(this.$refs.rename.value === ''){
                return alert('更改的歌单名不能为空')
            }


            let likeMusicList = JSON.parse(JSON.stringify(this.$store.state.myLikeMusic))


            let arr = JSON.parse(localStorage.getItem(likeMusicList[this.musicIndex].txt))
            localStorage.removeItem(likeMusicList[this.musicIndex].txt)
            localStorage.setItem( this.$refs.rename.value, JSON.stringify(arr) )

            // 新建歌单的项
            let likeMusic = {
                ...likeMusicList[this.musicIndex],
                query: {list: this.$refs.rename.value},
                txt: this.$refs.rename.value
            }

            likeMusicList.splice(this.musicIndex, 1 , likeMusic)

            this.$store.state.myLikeMusic = likeMusicList
            
            this.showRepeatName = false

        
        },
        getElementTop(elem, direction) {
            var elemTop = elem["offset" + direction]; //获得elem元素距相对定位的父元素的top
            elem = elem.offsetParent; //将elem换成起相对定位的父元素
            while (elem != null) {
                //只要还有相对定位的父元素
                //获得父元素 距他父元素的top值,累加到结果中
                elemTop += elem["offset" + direction]; //再次将elem换成他相对定位的父元素上;
                elem = elem.offsetParent;
            }
            return elemTop;
        },
        requstSongsheet(id) {
            let arr = []
            this.axios.get('/musicinfo/plist/list/' + id + '?json=true')
                .then(res => {
                    res.data.list.list.info.forEach(item => {
                        this.axios.get('player/yy/index.php?r=play/getdata&hash=' + item.hash)
                            .then(res => {
                                arr.push(res.data.data)
                            })
                    });
                })
            setTimeout(() => {
                this.$store.state.currentPlayerIndex = 0
                this.$store.state.playerList = arr
                this.$store.commit('searchMusic')

                this.$store.commit('pushHistoryMusic')
            }, 200)
                
        },
    },
    computed: {
        // 收藏歌单
        collectingSongs(){
            return this.$store.state.collectingSongs
        },
        // 自建歌单
        myLikeMusic(){
            return this.$store.state.myLikeMusic
        },
        // 我喜欢
        songList(){
            return this.$store.state.songList
        }
    },
    
}
</script>

<style scoped>
.menu {
    float: left;
    width: 190px;
    height: 590px;
    overflow-y: scroll;
    background: linear-gradient(to right, #f3f3f3, #eee);
}
.menu .column p {
    padding: 10px 10px 5px 15px;
    font-size: 12px;
    color: #999;
}
.menu .column p a {
    float: right;
    font-size: 12px;
    color: #666;
}
.menu .column p a i {
    color: #666;
    transition: 0.3s;
    opacity: 0.6;
    margin-right: 5px;
    font-size: 15px;
    vertical-align: middle;
}
.menu .column p a i:hover {
    opacity: 1;
}

.menu .column ul {
    margin-bottom: 10px;
}
.menu .column ul.hide {
    display: none;
}
.menu .column ul .addList{
    padding-left: 15px;
    height: 30px;
    line-height: 30px;
}

.menu .column ul a {
    display: block;
    width: 100%;
    padding-left: 15px;
    height: 30px;
    line-height: 30px;
    color: #333;
    box-sizing: border-box;
    font-size: 14px;
    font-weight: 400;
    white-space: nowrap;
    text-overflow: ellipsis;
    overflow: hidden;
}

.menu .column ul .active i {
    opacity: 1;
}
.menu .column ul  i {
    opacity: 0.6;
    font-size: 14px;
    line-height: 14px;
    margin-right: 5px;
    color: #000;
    vertical-align: middle;
}
.menu .column ul  i.icon-xin {
    color: #f00;
    font-weight: bold;
}
.menu .column ul a:not(.router-link-exact-active):hover {
    background-color: #fff;
}


/* 右键更多菜单 */
.mouseMenu {
    position: absolute;
    top: 0;
    left: 0;
    height: 690px;
    width: 1000px;
    z-index: 11;
}
.mouseMenu .mMenu{
    position: absolute;
    z-index: 11;
    top: 0;
    left: 0;
    width: 95px;
    color: #333;
    background-color: #fff;
    border: 1px solid #e4e4e4;
    border-radius: 4px;
    box-shadow: 0 0 10px #ccc;
    box-sizing: border-box;
    padding: 5px 0;
}
.mouseMenu .mMenu a {
    display: block;
    position: relative;
    padding: 0 15px;
    font-size: 14px;
    height: 30px;
    line-height: 30px;
    color: #666;
    white-space: nowrap;
}
.mouseMenu .mMenu a:hover {
    background-color: #f3f3f3;
}
.mouseMenu .mMenu a.hide{
    display: none
}
.mouseMenu .mMenu a.show{
    display: block
}

.repeat-name{
    position: absolute;
    z-index: 11;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    margin: auto;
    width: 340px;
    height: 225px;
    color: #333;
    background-color: #fff;
    border: 1px solid #e4e4e4;
    border-radius: 4px;
    box-shadow: 0 0 10px #ccc;
    box-sizing: border-box;
    padding: 5px 0;
}
.repeat-name > p{
    position: relative;
    text-align: center;
}
.repeat-name > p a{
    position: absolute;
    left: 10px;
    color: #666;
}
.repeat-name .content{
    padding: 30px 40px;
}
.repeat-name .content input{
    width: 100%;
    height: 35px;
    line-height: 35px;
    box-sizing: border-box;
    padding-left: 15px;
    margin: 10px 0 30px;
}
.repeat-name .content input::-webkit-input-placeholder{
    color: #999;
}
.repeat-name .content .btn{
    text-align: center;
}
.repeat-name .content .btn a{
    display: inline-block;
    width: 100px;
    height: 30px;
    line-height: 30px;
    font-size: 14px;
    border-radius: 4px;
}
.repeat-name .content .btn a:first-child{
    border: 1px solid #e4e4e4;
    color: #666;
}
.repeat-name .content .btn a:first-child:hover{
    border-color: #5aa0f8;
    color: #5aa0f8;
}
.repeat-name .content .btn a:last-child{
    border: 1px solid #5aa0f8;
    background-color: #5aa0f8;
    color: #fff;
}

.router-link-exact-active.router-link-active{
    background-color: #ddd;
    cursor: default;
}
</style>
