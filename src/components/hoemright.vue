<template>
      <div>
        <div>
          <div :style="xs||sm?{'display':'none'}:{'font-size':'4rem'}" class="leleo-left-welcome">{{ configdata.welcometitle }}</div>
        </div>
        <div>
          <v-row align="center">
            <v-col cols="12" md="8">
				<v-text-field class="v-card  search-field"
					:style="xs||sm?{'display':'none'}:{}"
					v-model="searchQuery"
					placeholder="搜索..."
					variant="outlined"
					rounded
					hide-details="true"
					@keyup.enter="performSearch"
					>
					<template v-slot:prepend-inner>
						<v-menu>
							<template v-slot:activator="{ props }">
							<v-btn
								variant="text"
								v-bind="props"
								class="engine-btn"
							>
								{{ selectedEngine.title }}
								<v-icon icon="mdi-chevron-down"></v-icon>
							</v-btn>
							</template>
							<v-list class="glass-list">
								<v-list-item
									v-for="engine in searchEngines"
									:key="engine.value"
									@click="selectedEngine = engine"
									density="compact"
								>
									{{ engine.title }}
								</v-list-item>
							</v-list>
						</v-menu>
					</template>

					<template v-slot:append-inner>
						<v-btn
						:icon="isUrl ? 'mdi-earth' : 'mdi-magnify'"
						variant="text"
						@click="performSearch"
						></v-btn>
					</template>
					</v-text-field>
            	<typewriter class="ma-3 d-flex align-center justify-center" style="min-height: 200px;"></typewriter>
            </v-col>
            <v-col cols="12" md="4" align="center">
              <v-card class="ma-3" hover
                >
                  <template v-slot:title >
                    <span class="leleo-card-title clock-font">{{formattedTime}}</span>
                  </template>
                  <template v-slot:subtitle>
                    <span style="font-weight: bold;">{{formattedDate}}</span>
                  </template>
                  <turntable :color1="configdata.color.turntablecolor1" :color2="configdata.color.turntablecolor2" />
              </v-card>
            </v-col>
          </v-row>
          
          <div class="website-section-title mt-3 ml-3">
            <v-chip prepend-icon="mdi-webhook" size="large" style="color: var(--leleo-vcard-color);">常用网站</v-chip>
          </div>

          <v-container>
            <div class="website-grid">
              <v-card v-for="(item,index) in websites" :key="item.url + index" class="website-tile" draggable="true"
                @dragstart="startDrag(index, $event)" @dragover.prevent @drop="dropWebsite(index)" @dragend="finishDrag" @click="openWebsite(item)">
                <v-menu location="bottom end">
                  <template v-slot:activator="{ props }">
                    <v-btn v-bind="props" icon="mdi-dots-horizontal" size="x-small" variant="flat" class="website-more" @click.stop></v-btn>
                  </template>
                  <v-list density="compact" class="website-menu">
                    <v-list-item prepend-icon="mdi-pencil-outline" title="重命名" @click="openRenameDialog(index)"></v-list-item>
                    <v-list-item prepend-icon="mdi-delete-outline" title="删除" class="delete-menu-item" @click="removeWebsite(index)"></v-list-item>
                  </v-list>
                </v-menu>
                <div class="website-image-wrap">
                  <v-img :src="item.img" :alt="item.title" class="website-image" contain>
                    <template v-slot:error>
                      <div class="website-fallback">{{ item.title.slice(0, 1).toUpperCase() }}</div>
                    </template>
                  </v-img>
                </div>
                <div class="website-name">{{ item.title }}</div>
              </v-card>
              <button type="button" class="website-add-tile" @click="openAddDialog">
                <span class="website-add-icon"><v-icon icon="mdi-plus"></v-icon></span>
                <span>添加快捷方式</span>
              </button>
            </div>
          </v-container>

          <v-dialog v-model="addDialog" width="480">
            <v-card class="pa-5" rounded="xl">
              <v-card-title>添加常用网站</v-card-title>
              <v-text-field v-model="newWebsite.title" label="网站名称" variant="outlined"></v-text-field>
              <v-text-field v-model="newWebsite.url" label="网站地址" placeholder="https://example.com" variant="outlined"></v-text-field>
              <v-text-field v-model="newWebsite.img" label="图片地址（可选）" hint="留空时自动使用网站图标" persistent-hint variant="outlined"></v-text-field>
              <v-alert v-if="formError" type="warning" variant="tonal" density="compact" class="mb-3">{{ formError }}</v-alert>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn @click="addDialog = false">取消</v-btn>
                <v-btn color="primary" variant="flat" @click="addWebsite">添加</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>

          <v-dialog v-model="renameDialog" width="420">
            <v-card class="pa-5" rounded="xl">
              <v-card-title>重命名快捷方式</v-card-title>
              <v-text-field v-model="renameTitle" label="新名称" maxlength="30" variant="outlined" autofocus @keyup.enter="renameWebsite"></v-text-field>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn @click="renameDialog = false">取消</v-btn>
                <v-btn color="primary" variant="flat" @click="renameWebsite">保存</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          
        </div>       
      </div>
</template> 

<script>
import typewriter from '../components/typewriter.vue';
import turntable from '../components/turntable.vue';
import { useDisplay } from 'vuetify'

export default {
    components: {
        typewriter,turntable
    },
    props: ['configdata','formattedTime','formattedDate','projectcards'],
	data() {
		return {
			websites: [],
			dragIndex: null,
			wasDragging: false,
			addDialog: false,
			renameDialog: false,
			renameIndex: -1,
			renameTitle: '',
			formError: '',
			newWebsite: { title: '', url: '', img: '' },
			searchQuery: '',
			selectedEngine: { title: 'Bing', value: 'bing' },
      		searchEngines :[
				{ title: 'Bing', value: 'bing' },
				{ title: 'Google', value: 'google' },
				{ title: '百度', value: 'baidu' },
				{ title: 'Yandex', value: 'yandex' },
				{ title: 'DuckDuckGo', value: 'duckduckgo' },
			]
		}
  },
	watch: {
		projectcards: {
			immediate: true,
			handler(value){
				if(!Array.isArray(value)) return;
				const saved = localStorage.getItem('xiaozhou-websites');
				try {
					this.websites = saved ? JSON.parse(saved) : value.map(item => ({ title: item.title, url: item.url, img: item.img }));
				} catch {
					this.websites = value.map(item => ({ title: item.title, url: item.url, img: item.img }));
				}
			}
		}
	},
    setup() {
      const { xs,sm,md } = useDisplay();
      return {xs,sm,md};
    },
	computed: {	
		isUrl(){
			const str = this.searchQuery.trim();
  			return this.isLikelyUrl(str);
		}
	},
    methods:{
	  persistWebsites(){
		localStorage.setItem('xiaozhou-websites', JSON.stringify(this.websites));
	  },
	  openAddDialog(){
		this.newWebsite = { title: '', url: '', img: '' };
		this.formError = '';
		this.addDialog = true;
	  },
	  addWebsite(){
		const title = this.newWebsite.title.trim();
		let url = this.newWebsite.url.trim();
		if(!title || !url){
		  this.formError = '请填写网站名称和地址。';
		  return;
		}
		if(!/^https?:\/\//i.test(url)) url = 'https://' + url;
		try {
		  const parsed = new URL(url);
		  const img = this.newWebsite.img.trim() || `${parsed.origin}/favicon.ico`;
		  this.websites.push({ title, url: parsed.href, img });
		  this.persistWebsites();
		  this.addDialog = false;
		} catch {
		  this.formError = '网站地址格式不正确。';
		}
	  },
	  removeWebsite(index){
		this.websites.splice(index, 1);
		this.persistWebsites();
	  },
	  openRenameDialog(index){
		this.renameIndex = index;
		this.renameTitle = this.websites[index]?.title || '';
		this.renameDialog = true;
	  },
	  renameWebsite(){
		const title = this.renameTitle.trim();
		if(!title || this.renameIndex < 0) return;
		this.websites[this.renameIndex].title = title;
		this.persistWebsites();
		this.renameDialog = false;
	  },
	  startDrag(index, event){
		this.dragIndex = index;
		this.wasDragging = true;
		event.dataTransfer.effectAllowed = 'move';
		event.dataTransfer.setData('text/plain', String(index));
	  },
	  dropWebsite(index){
		if(this.dragIndex === null || this.dragIndex === index) return;
		const [moved] = this.websites.splice(this.dragIndex, 1);
		this.websites.splice(index, 0, moved);
		this.persistWebsites();
		this.dragIndex = index;
	  },
	  finishDrag(){
		this.dragIndex = null;
		setTimeout(() => { this.wasDragging = false; }, 80);
	  },
	  openWebsite(item){
		if(this.wasDragging) return;
		window.open(item.url, '_blank', 'noopener,noreferrer');
	  },
	  performSearch() {
		const query = this.searchQuery.trim();
		if (!query) return;

		if (this.isUrl) {
			let url = query;
			// 自动补全协议（如果缺少）
			if (!/^[a-z]+:\/\//i.test(url)) {
				url = 'http://' + url; // 默认用http
			}
			
			window.open(url, '_blank');
		} else {
			const engineUrls = {
				google: `https://www.google.com/search?q=${encodeURIComponent(query)}`,
				bing: `https://www.bing.com/search?q=${encodeURIComponent(query)}`,
				baidu: `https://www.baidu.com/s?wd=${encodeURIComponent(query)}`,
				yandex: `https://yandex.com/search/?text=${encodeURIComponent(query)}`,
				duckduckgo: `https://duckduckgo.com/?q=${encodeURIComponent(query)}`
			};
			window.open(engineUrls[this.selectedEngine.value], '_blank');
		}
	  },
	  isLikelyUrl(input) {
		// 移除首尾空格
		const str = input.trim();
		
		// 情况1：明确包含协议头（http/https/ftp等）
		if (/^(https?|ftp):\/\//i.test(str)) return true;
		
		// 情况2：符合域名格式（支持国际化域名）
		const domainPattern = /^([a-z0-9-]+\.)+[a-z]{2,}(\/.*)?$/i;
		
		// 情况3：localhost或IP地址
		const localPattern = /^(localhost|\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})(:\d+)?(\/.*)?$/i;
		
		
		return (
			domainPattern.test(str) || 
			localPattern.test(str)
		);
		}
    }
};
</script>

<style scoped>
@import url(/css/app.less);
@import url(/css/mobile.less);
.search-field {
	border-radius: 22px !important;
	overflow: hidden;
}

.glass-list {
	background: transparent !important;
	backdrop-filter: blur(var(--leleo-blur));
	border-radius: 5%;
	color: var(--leleo-vcard-color);
	overflow: hidden;
}

.website-section-title {
	display: flex;
	align-items: center;
	justify-content: space-between;
	gap: 1rem;
}

.website-grid {
	display: flex;
	flex-wrap: wrap;
	gap: 14px 12px;
	align-items: flex-start;
}

.website-tile {
	position: relative;
	width: 72px;
	height: 72px;
	padding: 6px 6px 4px;
	text-align: center;
	cursor: pointer;
	transition: transform .2s ease, box-shadow .2s ease;
	overflow: visible;
}

.website-tile:hover {
	transform: translateY(-4px) scale(1.02);
	box-shadow: 0 10px 26px rgba(34, 28, 80, .2);
}

.website-image-wrap {
	width: 44px;
	height: 44px;
	margin: 0 auto;
	border-radius: 10px;
	overflow: hidden;
	display: flex;
	align-items: center;
	justify-content: center;
	background: rgba(255, 255, 255, .38);
}

.website-image {
	width: 78%;
	height: 78%;
}

.website-fallback {
	width: 100%;
	height: 100%;
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 2rem;
	font-weight: 800;
	background: linear-gradient(145deg, #7c5cfc, #32e6c4);
	color: white;
}

.website-name {
	padding: 3px 1px 0;
	font-weight: 700;
	font-size: .68rem;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.website-more {
	position: absolute;
	top: -8px;
	right: -8px;
	z-index: 5;
	opacity: 0;
	transform: scale(.8);
	transition: opacity .16s ease, transform .16s ease;
	background: rgba(255,255,255,.96) !important;
	box-shadow: 0 3px 12px rgba(20,25,40,.22);
}

.website-tile:hover .website-more,
.website-more:focus-visible {
	opacity: 1;
	transform: scale(1);
}

.website-menu {
	min-width: 132px;
	background: rgba(255,255,255,.97) !important;
	box-shadow: 0 10px 30px rgba(20,25,40,.2);
}

.delete-menu-item { color: #c74646; }

.website-add-tile {
	width: 82px;
	min-height: 72px;
	border: 0;
	background: transparent;
	color: var(--leleo-vcard-color);
	cursor: pointer;
	display: flex;
	flex-direction: column;
	align-items: center;
	gap: 4px;
	font: inherit;
	font-size: .68rem;
	font-weight: 700;
}

.website-add-icon {
	width: 44px;
	height: 44px;
	display: grid;
	place-items: center;
	border-radius: 12px;
	background: rgba(255,255,255,.94);
	color: #30333b;
	box-shadow: 0 4px 14px rgba(28,28,40,.16);
	transition: transform .18s ease, box-shadow .18s ease;
}

.website-add-tile:hover .website-add-icon {
	transform: translateY(-3px);
	box-shadow: 0 8px 20px rgba(28,28,40,.22);
}

@media (max-width: 700px) {
	.website-grid { gap: 8px; }
	.website-more { opacity: 1; transform: scale(.9); }
}
</style>
