<!-- 模板 -->
<template>
	<div id="home">
		<AppHeader
			v-on:titleChanged="updateTitle($event)"
			v-bind:title="title"
		></AppHeader>
		<Users v-bind:users="users"></Users>
		<Users v-bind:users="users"></Users>
		<AppFooter v-bind:title="title"> </AppFooter>
	</div>
</template>

<!--行为 -->
<script>
import Users from "./Users";
import AppHeader from "./Header";
import AppFooter from "./Footer";
export default {
	name: "Home",
	data() {
		return {
			users: [
				{ name: "henry", position: "web", show: false, key: 0 },
				{ name: "henry", position: "web", show: false, key: 1 },
				{ name: "henry", position: "web", show: false, key: 2 },
				{ name: "henry", position: "web", show: false, key: 3 },
				{ name: "henry", position: "web", show: false, key: 4 },
			],
			title: "title数据来自 app.vue",
		};
	},
	methods: {
		updateTitle(title_) {
			this.title = title_;
		},
	},
	components: {
		Users,
		AppHeader,
		AppFooter,
	},
};
</script>

<!-- 样式 -->
<style scoped>
h1 {
	color: black;
}
</style>
