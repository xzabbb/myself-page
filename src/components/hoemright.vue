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
            <v-btn size="small" variant="tonal" prepend-icon="mdi-shape-outline" class="mr-4" @click="categoryDialog = true">管理分类</v-btn>
          </div>

          <v-container>
            <v-chip-group v-model="activeCategoryId" mandatory class="category-tabs mb-4">
              <v-chip v-for="(category, categoryIndex) in categories" :key="category.id" :value="category.id" filter variant="tonal"
                draggable="true" class="category-chip" :class="{ 'category-dragging': categoryDragIndex === categoryIndex }"
                @dragstart.stop="startCategoryDrag(categoryIndex, $event)" @dragover.prevent @drop.stop.prevent="dropCategory(categoryIndex)" @dragend="finishCategoryDrag">
                {{ category.name }}
                <span class="category-count">{{ countByCategory(category.id) }}</span>
              </v-chip>
            </v-chip-group>
            <div class="website-grid">
              <v-card v-for="entry in currentWebsites" :key="entry.item.url + entry.index" class="website-tile" draggable="true"
                @dragstart="startDrag(entry.index, $event)" @dragover.prevent @drop="dropWebsite(entry.index)" @dragend="finishDrag" @click="openWebsite(entry.item)">
                <v-tooltip activator="parent" location="top" :open-delay="250">{{ entry.item.title }}</v-tooltip>
                <v-menu location="bottom end">
                  <template v-slot:activator="{ props }">
                    <v-btn v-bind="props" icon="mdi-dots-horizontal" size="x-small" variant="flat" class="website-more" @click.stop></v-btn>
                  </template>
                  <v-list density="compact" class="website-menu">
                    <v-list-item prepend-icon="mdi-pencil-outline" title="编辑" @click="openEditDialog(entry.index)"></v-list-item>
                    <v-list-item prepend-icon="mdi-delete-outline" title="删除" class="delete-menu-item" @click="removeWebsite(entry.index)"></v-list-item>
                  </v-list>
                </v-menu>
                <div class="website-image-wrap">
                  <v-img :src="entry.item.img" :alt="entry.item.title" class="website-image" contain>
                    <template v-slot:error>
                      <div class="website-fallback">{{ entry.item.title.slice(0, 1).toUpperCase() }}</div>
                    </template>
                  </v-img>
                </div>
                <div class="website-name">{{ entry.item.title }}</div>
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
              <v-select v-model="newWebsite.categoryId" :items="categories" item-title="name" item-value="id" label="所属分类" variant="outlined"></v-select>
              <v-alert v-if="formError" type="warning" variant="tonal" density="compact" class="mb-3">{{ formError }}</v-alert>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn @click="addDialog = false">取消</v-btn>
                <v-btn color="primary" variant="flat" @click="addWebsite">添加</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>

          <v-dialog v-model="editDialog" width="480">
            <v-card class="pa-5" rounded="xl">
              <v-card-title>编辑快捷方式</v-card-title>
              <v-text-field v-model="editWebsite.title" label="网站名称" maxlength="30" variant="outlined" autofocus></v-text-field>
              <v-text-field v-model="editWebsite.url" label="网站地址" variant="outlined"></v-text-field>
              <v-text-field v-model="editWebsite.img" label="图片地址" variant="outlined"></v-text-field>
              <v-select v-model="editWebsite.categoryId" :items="categories" item-title="name" item-value="id" label="所属分类" variant="outlined"></v-select>
              <v-alert v-if="editError" type="warning" variant="tonal" density="compact" class="mb-3">{{ editError }}</v-alert>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn @click="editDialog = false">取消</v-btn>
                <v-btn color="primary" variant="flat" @click="saveWebsiteEdit">保存</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>

          <v-dialog v-model="categoryDialog" width="500">
            <v-card class="pa-5" rounded="xl">
              <v-card-title>管理网站分类</v-card-title>
              <div class="category-manager-list">
                <div v-for="category in categories" :key="category.id" class="category-manager-row">
                  <v-text-field v-model="category.name" density="compact" variant="outlined" hide-details maxlength="12" @change="persistCategories"></v-text-field>
                  <v-btn icon="mdi-delete-outline" variant="text" color="error" :disabled="categories.length === 1" title="删除分类" @click="removeCategory(category.id)"></v-btn>
                </div>
              </div>
              <div class="category-add-row mt-4">
                <v-text-field v-model="newCategoryName" density="compact" variant="outlined" hide-details label="新分类名称" maxlength="12" @keyup.enter="addCategory"></v-text-field>
                <v-btn color="primary" variant="flat" prepend-icon="mdi-plus" @click="addCategory">添加</v-btn>
              </div>
              <v-card-actions class="mt-3">
                <v-spacer></v-spacer>
                <v-btn color="primary" variant="flat" @click="persistCategories(); categoryDialog = false">完成</v-btn>
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
			categories: [],
			activeCategoryId: 'study',
			categoryDialog: false,
			newCategoryName: '',
			categoryDragIndex: null,
			shortcutSyncReady: false,
			shortcutSyncTimer: null,
			dragIndex: null,
			wasDragging: false,
			addDialog: false,
			editDialog: false,
			editIndex: -1,
			editWebsite: { title: '', url: '', img: '', categoryId: 'study' },
			editError: '',
			formError: '',
			newWebsite: { title: '', url: '', img: '', categoryId: 'study' },
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
				const defaults = [
					{ id: 'vpn', name: '梯子' },
					{ id: 'study', name: '学习' },
					{ id: 'ip', name: 'IP' },
					{ id: 'entertainment', name: '娱乐' }
				];
				try {
					const savedCategories = JSON.parse(localStorage.getItem('xiaozhou-website-categories'));
					this.categories = Array.isArray(savedCategories) && savedCategories.length ? savedCategories : defaults;
				} catch {
					this.categories = defaults;
				}
				const saved = localStorage.getItem('xiaozhou-websites');
				try {
					const parsed = saved ? JSON.parse(saved) : value.map(item => ({ title: item.title, url: item.url, img: item.img }));
					this.websites = parsed.map(item => ({ ...item, categoryId: item.categoryId || 'study' }));
				} catch {
					this.websites = value.map(item => ({ title: item.title, url: item.url, img: item.img, categoryId: 'study' }));
				}
				this.persistCategories();
				this.persistWebsites();
				this.$nextTick(() => this.loadRemoteShortcuts(true));
			}
		}
	},
	mounted(){
		this.shortcutSyncTimer = setInterval(() => this.loadRemoteShortcuts(false), 10000);
	},
	beforeUnmount(){
		if(this.shortcutSyncTimer) clearInterval(this.shortcutSyncTimer);
	},
    setup() {
      const { xs,sm,md } = useDisplay();
      return {xs,sm,md};
    },
	computed: {	
		isUrl(){
			const str = this.searchQuery.trim();
  			return this.isLikelyUrl(str);
		},
		currentWebsites(){
			return this.websites.map((item, index) => ({ item, index })).filter(entry => entry.item.categoryId === this.activeCategoryId);
		}
	},
    methods:{
	  persistWebsites(){
		localStorage.setItem('xiaozhou-websites', JSON.stringify(this.websites));
		if(this.shortcutSyncReady) this.pushRemoteShortcuts();
	  },
	  persistCategories(){
		this.categories = this.categories.filter(category => category.name.trim()).map(category => ({ ...category, name: category.name.trim() }));
		localStorage.setItem('xiaozhou-website-categories', JSON.stringify(this.categories));
		if(this.shortcutSyncReady) this.pushRemoteShortcuts();
	  },
	  async pushRemoteShortcuts(){
		try {
			await fetch('https://xiaozhou-embedded-workbench.tealbowl14.chatgpt.site/api/shortcuts', {
				method: 'PUT',
				headers: { 'content-type': 'application/json' },
				body: JSON.stringify({ data: { categories: this.categories, websites: this.websites } })
			});
		} catch (error) {
			console.error('常用网站同步失败', error);
		}
	  },
	  async loadRemoteShortcuts(seedWhenEmpty = false){
		try {
			const response = await fetch('https://xiaozhou-embedded-workbench.tealbowl14.chatgpt.site/api/shortcuts', { cache: 'no-store' });
			if(!response.ok) throw new Error(`同步请求失败: ${response.status}`);
			const result = await response.json();
			if(result.data?.categories?.length && Array.isArray(result.data.websites)){
				const remoteSnapshot = JSON.stringify(result.data);
				const localSnapshot = JSON.stringify({ categories: this.categories, websites: this.websites });
				if(seedWhenEmpty && this.websites.length > result.data.websites.length){
					await this.pushRemoteShortcuts();
				} else if(remoteSnapshot !== localSnapshot){
					this.categories = result.data.categories;
					this.websites = result.data.websites;
					if(!this.categories.some(category => category.id === this.activeCategoryId)) this.activeCategoryId = this.categories[0].id;
					localStorage.setItem('xiaozhou-website-categories', JSON.stringify(this.categories));
					localStorage.setItem('xiaozhou-websites', JSON.stringify(this.websites));
				}
			} else if(seedWhenEmpty) {
				await this.pushRemoteShortcuts();
			}
			this.shortcutSyncReady = true;
		} catch (error) {
			console.error('读取常用网站同步数据失败', error);
			this.shortcutSyncReady = true;
		}
	  },
	  countByCategory(categoryId){
		return this.websites.filter(item => item.categoryId === categoryId).length;
	  },
	  startCategoryDrag(index, event){
		this.categoryDragIndex = index;
		event.dataTransfer.effectAllowed = 'move';
		event.dataTransfer.setData('text/plain', this.categories[index].id);
	  },
	  dropCategory(index){
		if(this.categoryDragIndex === null || this.categoryDragIndex === index) return;
		const [movedCategory] = this.categories.splice(this.categoryDragIndex, 1);
		this.categories.splice(index, 0, movedCategory);
		this.categoryDragIndex = index;
		this.persistCategories();
	  },
	  finishCategoryDrag(){
		this.categoryDragIndex = null;
	  },
	  openAddDialog(){
		this.newWebsite = { title: '', url: '', img: '', categoryId: this.activeCategoryId };
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
		  this.websites.push({ title, url: parsed.href, img, categoryId: this.newWebsite.categoryId || this.activeCategoryId });
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
	  openEditDialog(index){
		this.editIndex = index;
		this.editWebsite = { ...this.websites[index] };
		this.editError = '';
		this.editDialog = true;
	  },
	  saveWebsiteEdit(){
		const title = this.editWebsite.title.trim();
		let url = this.editWebsite.url.trim();
		if(!title || !url || this.editIndex < 0){ this.editError = '请填写网站名称和地址。'; return; }
		if(!/^https?:\/\//i.test(url)) url = 'https://' + url;
		try {
		  const parsed = new URL(url);
		  this.websites[this.editIndex] = { ...this.editWebsite, title, url: parsed.href, img: this.editWebsite.img.trim() || `${parsed.origin}/favicon.ico` };
		  this.persistWebsites();
		  this.editDialog = false;
		} catch { this.editError = '网站地址格式不正确。'; }
	  },
	  addCategory(){
		const name = this.newCategoryName.trim();
		if(!name || this.categories.some(category => category.name === name)) return;
		const id = `category-${Date.now()}`;
		this.categories.push({ id, name });
		this.newCategoryName = '';
		this.persistCategories();
		this.activeCategoryId = id;
	  },
	  removeCategory(categoryId){
		if(this.categories.length <= 1) return;
		const fallback = this.categories.find(category => category.id !== categoryId);
		this.websites.forEach(item => { if(item.categoryId === categoryId) item.categoryId = fallback.id; });
		this.categories = this.categories.filter(category => category.id !== categoryId);
		if(this.activeCategoryId === categoryId) this.activeCategoryId = fallback.id;
		this.persistCategories();
		this.persistWebsites();
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

.category-tabs {
	padding: 3px 6px;
	border-radius: 16px;
	background: rgba(255,255,255,.16);
}

.category-tabs :deep(.v-chip),
.category-tabs :deep(.v-chip__content),
.category-tabs :deep(.v-chip__filter),
.website-section-title :deep(.v-btn),
.website-section-title :deep(.v-icon) {
	color: #fff !important;
}

.category-chip {
	cursor: grab;
	user-select: none;
	transition: transform .18s ease, opacity .18s ease;
}

.category-chip:active { cursor: grabbing; }
.category-chip.category-dragging {
	opacity: .58;
	transform: scale(.95);
}

.category-count {
	margin-left: 7px;
	min-width: 18px;
	height: 18px;
	padding: 0 5px;
	display: inline-grid;
	place-items: center;
	border-radius: 9px;
	font-size: .66rem;
	background: rgba(255,255,255,.44);
	color: #fff !important;
}

.category-manager-list {
	display: grid;
	gap: 10px;
}

.category-manager-row,
.category-add-row {
	display: flex;
	align-items: center;
	gap: 10px;
}

.category-manager-row .v-text-field,
.category-add-row .v-text-field {
	flex: 1;
}

.website-tile {
	position: relative;
	width: 96px;
	height: 96px;
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
	width: 66px;
	height: 66px;
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
	width: 96px;
	min-height: 96px;
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
	width: 66px;
	height: 66px;
	display: grid;
	place-items: center;
	border-radius: 10px;
	background: rgba(255, 255, 255, .38);
	color: #30333b;
	box-shadow: none;
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
