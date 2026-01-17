<script setup lang="ts">
import type { Comment } from '@weibo-archiver/core'
import { formatDate, formatNumber } from '@weibo-archiver/core'
import { Heart, MessageCircle } from 'lucide-vue-next'
import { computed, ref, watch } from 'vue'
import { emitter } from '@/composables'
import Avatar from '../common/Avatar.vue'
import LazyImage from '../common/LazyImage.vue'
import { WeiboText } from './WeiboText'

const props = defineProps<{
  comments: Comment[]
}>()

const pageSize = 10
const currentPage = ref(1)

const totalPages = computed(() => Math.max(1, Math.ceil(props.comments.length / pageSize)))
const pagedComments = computed(() => {
  const start = (currentPage.value - 1) * pageSize
  return props.comments.slice(start, start + pageSize)
})

watch(
  () => props.comments,
  () => {
    currentPage.value = 1
  },
)

function previewImage(url: string) {
  emitter.emit('open-image-preview', {
    imgs: [url],
    index: 0,
  })
}

function goToPage(page: number) {
  currentPage.value = Math.min(Math.max(page, 1), totalPages.value)
}
</script>

<template>
  <ul
    class="list"
  >
    <li
      v-for="comment in pagedComments"
      :key="comment.id"
      class="flex gap-3 py-2"
    >
      <Avatar
        :src="comment.user.avatar"
        :alt="comment.user.name"
        :size="8"
      />
      <div class="flex flex-col w-full">
        <a
          :href="`https://weibo.com/n/${comment.user.name}`"
          target="_blank"
          class="font-bold text-base-content"
        >
          {{ comment.user.name }}
        </a>

        <WeiboText
          class="my-1 text-base"
          :text="comment.text"
        />

        <LazyImage
          v-if="comment.img"
          :src="comment.img"
          class="h-42 w-fit mt-2 rounded-md"
          @click="previewImage(comment.img)"
        />

        <div class=" flex items-center gap-2 mt-2 ml-auto text-xs text-gray-500">
          <time>
            发布于 {{ formatDate(comment.createdAt) }}
          </time>
          <div>
            {{ comment.regionName }}
          </div>

          <button
            :title="`点赞 ${comment.likesCount}`"
            class="ml-1 flex items-center gap-1 hover:text-red-500"
          >
            <Heart class="w-4 h-4" />
            {{ formatNumber(comment.likesCount) }}
          </button>
          <button
            :title="`评论 ${comment.commentsCount}`"
            class="flex items-center gap-1 hover:text-blue-500"
          >
            <MessageCircle class="w-4 h-4" />
            {{ formatNumber(comment.commentsCount) }}
          </button>
        </div>
      </div>
    </li>
  </ul>

  <div
    v-if="comments.length > pageSize"
    class="mt-2 flex items-center justify-end gap-2 text-sm text-gray-500"
  >
    <button
      class="btn btn-ghost btn-xs"
      :disabled="currentPage === 1"
      @click="goToPage(currentPage - 1)"
    >
      上一页
    </button>
    <span>
      第 {{ currentPage }} / {{ totalPages }} 页
    </span>
    <button
      class="btn btn-ghost btn-xs"
      :disabled="currentPage === totalPages"
      @click="goToPage(currentPage + 1)"
    >
      下一页
    </button>
  </div>
</template>
