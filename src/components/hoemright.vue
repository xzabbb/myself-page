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
            <div class="website-actions">
              <v-btn size="small" variant="flat" color="primary" prepend-icon="mdi-plus" title="添加网站" @click="openAddDialog">添加</v-btn>
              <v-btn size="small" variant="flat" :color="deleteMode ? 'success' : 'error'" :prepend-icon="deleteMode ? 'mdi-check' : 'mdi-minus'" title="删除网站" @click="deleteMode = !deleteMode">{{ deleteMode ? '完成' : '删除' }}</v-btn>
            </div>
          </div>

          <v-container>
            <div class="website-grid">
              <v-card v-for="(item,index) in websites" :key="item.url + index" class="website-tile" @click="openWebsite(item)">
                <v-btn v-if="deleteMode" icon="mdi-close" size="x-small" color="error" class="website-delete" @click.stop="removeWebsite(index)"></v-btn>
                <div class="website-image-wrap">
                  <v-img :src="item.img" :alt="item.title" class="website-image" contain>
                    <template v-slot:error>
                      <div class="website-fallback">{{ item.title.slice(0, 1).toUpperCase() }}</div>
                    </template>
                  </v-img>
                </div>
                <div class="website-name">{{ item.title }}</div>
              </v-card>
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
			deleteMode: false,
			addDialog: false,
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
		if(!this.websites.length) this.deleteMode = false;
	  },
	  openWebsite(item){
		if(this.deleteMode) return;
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

.website-actions {
	display: flex;
	gap: .55rem;
	margin-right: 1rem;
}

.website-grid {
	display: flex;
	flex-wrap: wrap;
	gap: 10px;
}

.website-tile {
	position: relative;
	width: 72px;
	height: 72px;
	padding: 6px 6px 4px;
	text-align: center;
	cursor: pointer;
	transition: transform .2s ease, box-shadow .2s ease;
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

.website-delete {
	position: absolute;
	top: 4px;
	right: 4px;
	z-index: 3;
}

@media (max-width: 700px) {
	.website-grid { gap: 8px; }
	.website-actions .v-btn { min-width: 34px; padding: 0 8px; }
}
</style>
