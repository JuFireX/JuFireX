<!-- @author: Churk -->
<!-- @status: 待完善 -->
<!-- @description: Profile 档案页 -->

<template>
  <ScrollContainer>
    <n-space vertical size="large">
      <n-h1>{{ $t("page.user.profile.title") }}</n-h1>

      <n-space justify="center">
        <n-grid cols="1 m:3" responsive="screen" x-gap="16" y-gap="16" style="max-width: 1500px">
          <!-- 左：用户信息卡片 -->
          <n-grid-item :span="1">
            <n-space vertical :size="18">
              <!-- 用户信息卡片 -->
              <n-card>
                <n-space vertical size="large" align="center">
                  <n-avatar round :size="120" :src="avatarUrl" />
                  <strong>{{ userStore.currentUser?.username || "-" }}</strong>
                  <n-tag type="info" :bordered="false">
                    {{ userStore.currentUser?.role_name || "-" }}
                  </n-tag>
                </n-space>
              </n-card>

              <!-- 统计卡片 -->
              <n-card>
                <n-spin :show="statsLoading" style="width: 100%">
                  <n-grid :cols="3" x-gap="8">
                    <n-grid-item>
                      <n-space vertical size="small" align="center">
                        <strong>{{ stats.blogs }}</strong>
                        <span>博客</span>
                      </n-space>
                    </n-grid-item>
                    <n-grid-item>
                      <n-space vertical size="small" align="center">
                        <strong>{{ stats.navigations }}</strong>
                        <span>导航</span>
                      </n-space>
                    </n-grid-item>
                    <n-grid-item>
                      <n-space vertical size="small" align="center">
                        <strong>{{ stats.projects }}</strong>
                        <span>项目</span>
                      </n-space>
                    </n-grid-item>
                  </n-grid>
                </n-spin>
              </n-card>

              <!-- 外链卡片 -->
              <n-card title="外链">
                <n-space vertical>
                  <n-button block secondary>GitHub（占位）</n-button>
                  <n-button block secondary>个人网站（占位）</n-button>
                  <n-button block secondary>X / Twitter（占位）</n-button>
                </n-space>
              </n-card>

              <!-- 仅自己可见卡片 -->
              <n-card title="仅自己可见">
                <n-space vertical size="medium">
                  <n-alert type="info" :show-icon="false"> 此区域仅你自己可见 </n-alert>
                  <n-descriptions label-placement="left" bordered :column="1" size="small">
                    <n-descriptions-item label="Email">
                      {{ userStore.currentUser?.email }}
                    </n-descriptions-item>
                    <n-descriptions-item label="UUID">
                      {{ userStore.currentUser?.uuid }}
                    </n-descriptions-item>
                    <n-descriptions-item label="Joined At">
                      {{ formatDate(userStore.currentUser?.created_at) }}
                    </n-descriptions-item>
                  </n-descriptions>
                </n-space>
              </n-card>
            </n-space>
          </n-grid-item>

          <!-- 右：贡献墙和README卡片 -->
          <n-grid-item :span="2">
            <n-space vertical :size="18">
              <!-- 贡献墙卡片 -->
              <n-card title="贡献墙">
                <n-space vertical size="medium">
                  <n-alert type="default" :show-icon="false"> 先使用模拟贡献数据，后续可替换为真实行为数据。 </n-alert>
                  <n-heatmap :data="contributionData" size="small" :tooltip="true" :show-week-labels="false">
                    <template #tooltip="{ timestamp, value }">
                      <n-space vertical size="small">
                        <strong>{{ formatContributionDate(timestamp) }}</strong>
                        <span>贡献值：{{ value ?? 0 }}</span>
                      </n-space>
                    </template>
                  </n-heatmap>
                </n-space>
              </n-card>

              <!-- README卡片 (之后改造成bio签名卡片)-->
              <n-card title="README">
                <n-space justify="space-between" align="center">
                  <span>你可以在后续版本中编辑你的个人 README</span>
                  <n-button secondary>编辑 README（占位）</n-button>
                </n-space>
              </n-card>

              <!-- README内容卡片 -->
              <MarkdownContainer :content="profileReadmeContent" />
            </n-space>
          </n-grid-item>
        </n-grid>
      </n-space>
    </n-space>
  </ScrollContainer>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from "vue";
import { useUserStore } from "@/stores/user";
import { userService } from "@/services/user";
import { blogService } from "@/services/blog";
import { projectService } from "@/services/project";
import { navigationService } from "@/services/navigation";
import {
  NCard,
  NSpace,
  NFlex,
  NH1,
  NAvatar,
  NButton,
  NDescriptions,
  NDescriptionsItem,
  NGrid,
  NGridItem,
  NDivider,
  NTag,
  NAlert,
  NSpin,
  NHeatmap,
} from "naive-ui";
import { ScrollContainer } from "@/components/scroll-container";
import { MarkdownContainer } from "@/components/markdown-container";

const userStore = useUserStore();
const statsLoading = ref(false);
const stats = ref({
  blogs: 0,
  navigations: 0,
  projects: 0,
});
const profileReadmeContent = ref(`# 个人 README（占位）

你好，这里是你的个人主页 README 区域。

- 你可以介绍自己的方向和兴趣
- 可以放置代表作与链接
- 可以写近期计划与更新记录

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

测试高度
`);
const contributionData = computed(() => {
  const today = new Date();
  const days = 365;
  const seed = stats.value.blogs * 17 + stats.value.navigations * 11 + stats.value.projects * 23 + 7;
  const base = Math.max(2, stats.value.blogs + stats.value.navigations + stats.value.projects);

  return Array.from({ length: days }, (_, index) => {
    const date = new Date(today);
    date.setDate(today.getDate() - (days - 1 - index));

    const phase = Math.abs(Math.sin((index + seed) / 10));
    const weekendFactor = date.getDay() === 0 || date.getDay() === 6 ? 0.5 : 1;
    const value = Math.round(base * phase * weekendFactor);

    return {
      timestamp: new Date(date.getFullYear(), date.getMonth(), date.getDate()).getTime(),
      value: Math.min(value, 20),
    };
  });
});

const avatarUrl = computed(() => {
  if (!userStore.currentUser?.uuid) return "";
  return userService.getAvatarUrl(userStore.currentUser.uuid) + `?t=${userStore.avatarTimestamp}`;
});

function formatDate(dateStr?: string) {
  if (!dateStr) return "-";
  return new Date(dateStr).toLocaleString();
}

function formatContributionDate(timestamp: number) {
  return new Date(timestamp).toLocaleDateString();
}

async function fetchStats() {
  if (!userStore.currentUser?.uuid) return;
  statsLoading.value = true;
  try {
    const [blogs, navigations, projects] = await Promise.all([
      blogService.getAll(true),
      navigationService.getAll(true),
      projectService.getAll(true),
    ]);

    const currentUserUuid = userStore.currentUser.uuid;
    stats.value = {
      blogs: blogs.filter((item) => item.owner_uuid === currentUserUuid).length,
      navigations: navigations.filter((item) => item.owner_uuid === currentUserUuid).length,
      projects: projects.filter((item) => item.owner_uuid === currentUserUuid).length,
    };
  } catch (e) {
    console.error(e);
  } finally {
    statsLoading.value = false;
  }
}

onMounted(async () => {
  if (!userStore.currentUser) {
    await userStore.checkAuth();
  }
  await fetchStats();
});
</script>
