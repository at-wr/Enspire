<script lang="ts" setup>
import type { Club, Clubs } from '@@/types/clubs'
import Badge from '@/components/ui/badge/Badge.vue'
import { Button } from '@/components/ui/button'
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { Separator } from '@/components/ui/separator'
import sanitizeHtml from 'sanitize-html'
import { useRoute } from 'vue-router'

const { data } = await useFetch<Clubs>('/api/club/all_details')

const clubs = data.value!

const route = useRoute()
const id = route.params.id // Fetch current Club ID via route params

// Filter clubs based on C_GroupsID and include information at the same level as groups
const filteredClubs = Object.values(clubs).flatMap(clubCategory =>
  clubCategory.filter((club: Club) =>
    club.groups.some(group => group.C_GroupsID === id),
  ).map((club: Club) => ({
    ...club, // Spread to include all same-level information
    groups: club.groups.filter(group => group.C_GroupsID === id), // Filter groups to only include those that match the ID
  })),
) as Club[]

// Get the number of members in each group
const groupMemberCounts = filteredClubs.length > 0 ? filteredClubs[0].gmember.length : 0

// Get the Chinese Description of the club
let hasDescriptionC = false
let Description_C = ''

if (filteredClubs[0] && filteredClubs[0].groups[0].C_DescriptionC) {
  const tempDescription = sanitizeHtml(filteredClubs[0].groups[0].C_DescriptionC, {
    allowedTags: [],
  })
  if (tempDescription.trim() !== '') {
    Description_C = tempDescription
    hasDescriptionC = true
  }
}

definePageMeta({
  middleware: ['auth'],
  breadcrumb: '社团详情',
})

useHead({
  title: 'Clubs | Enspire',
})
</script>

<template>
  <div v-if="filteredClubs.length > 0">
    <div v-for="club in filteredClubs" :key="club.groups[0].C_GroupNo">
      <div v-for="group in club.groups" :key="group.C_GroupsID">
        <div class="flex flex-col-reverse xl:flex-row">
          <Card class="mt-2 w-full xl:mt-0 xl:w-3/4">
            <CardHeader>
              <CardTitle class="flex items-center justify-between">
                <div>
                  {{ group.C_NameC }}
                </div>
                <Badge v-if="club.gmember.length === 0" variant="destructive">
                  已解散
                </badge>
              </CardTitle>

              <CardDescription class="flex items-center">
                <Icon name="material-symbols:language" />
                <div v-if="group.C_NameE" class="ml-1">
                  {{ group.C_NameE }}
                </div>
                <div v-else class="ml-1">
                  Club Description
                </div>
              </CardDescription>
            </CardHeader>
            <CardContent>
              <div class="font-bold">
                简介
              </div>
              <div v-if="hasDescriptionC" class="my-3 text-sm" v-text="Description_C" />
              <div v-else class="my-2 w-full text-center text-sm text-muted-foreground italic">
                暂无简介 ;-(
              </div>
              <!-- Don't show the English Description until i18n is completed -->
              <Separator class="my-4" />
              <div class="font-bold">
                成员
              </div>
              <div v-if="club.gmember.length === 0" class="my-2 w-full text-center text-sm text-muted-foreground italic">
                暂无成员 ;-(
              </div>
              <div v-else class="mt-3">
                <div class="flex flex-wrap">
                  <div v-for="(member, index) in club.gmember" :key="member.StudentID" class="flex items-center">
                    <div class="mt-0.5 flex items-center text-sm">
                      <span class="">{{ member.S_Name }}</span>
                      <span v-if="member.S_Nickname" class="ml-1 text-muted-foreground">({{ member.S_Nickname }})</span>
                      <Badge v-if="Number(member.LeaderYes) === 2" variant="default" class="ml-1 -py-0.5">
                        社长
                      </Badge>
                      <Badge v-else-if="Number(member.LeaderYes) === 1" variant="secondary" class="ml-1 -py-0.5">
                        副社
                      </Badge>
                      <span v-if="index < club.gmember.length - 1" class="mx-2">/</span>
                    </div>
                  </div>
                </div>
              </div>
            </CardContent>
          </Card>
          <Card class="h-min w-full xl:ml-2 xl:w-1/4">
            <CardHeader>
              <CardTitle class="h-min flex items-center gap-x-1">
                社团属性
              </CardTitle>
              <CardDescription class="flex items-center">
                <Icon name="material-symbols:info-outline" />
                <div class="ml-1">
                  Club Information
                </div>
              </CardDescription>
            </CardHeader>
            <CardContent>
              <div>
                <span class="font-bold">社团类型</span>: {{ group.C_Category }}
              </div>
              <div>
                <span class="font-bold">社团人数</span>: {{ groupMemberCounts }} 人
              </div>
              <div v-if="club.supervisor" class="flex">
                <span class="font-bold">指导老师:</span>
                <span v-for="supervisor in club.supervisor" :key="supervisor.TeacherID" class="ml-2">
                  {{ supervisor.T_Name }} ({{ supervisor.T_Nickname }})
                </span>
              </div>
            </CardContent>
          </Card>
        </div>
        <!--        <div style="display:none"> -->
        <!--          <Card v-if="club.grecord.length > 0" class="w-full"> -->
        <!--            <CardHeader> -->
        <!--              <CardTitle class="flex justify-between items-center"> -->
        <!--                <div> -->
        <!--                  近期活动 -->
        <!--                </div> -->
        <!--              </CardTitle> -->

        <!--              <CardDescription class="flex items-center"> -->
        <!--                <Icon name="material-symbols:draw-outline" /> -->
        <!--                <div class="ml-1"> -->
        <!--                  Recent Activities -->
        <!--                </div> -->
        <!--              </CardDescription> -->
        <!--            </CardHeader> -->
        <!--            <CardContent> -->
        <!--              <div v-for="grecord in club.grecord" :key="grecord.C_Theme"> -->
        <!--                <div class="font-bold"> -->
        <!--                  {{ grecord.C_Theme }} -->
        <!--                </div> -->
        <!--                <div class="text-sm text-muted-foreground mb-1"> -->
        <!--                  <Icon name="material-symbols:schedule-outline" /> -->
        <!--                  {{ grecord.C_Date }} -->
        <!--                </div> -->
        <!--                <div class="text-sm"> -->
        <!--                  {{ grecord.C_Reflection }} -->
        <!--                </div> -->
        <!--                <br> -->
        <!--              </div> -->
        <!--            </CardContent> -->
        <!--          </Card> -->
        <!--        </div> -->
      </div>
    </div>
  </div>
  <div v-else>
    <div class="h-1/2 flex flex-col justify-center text-center">
      <h3 class="text-xl font-bold">
        你当前访问的页面不存在，也许你应该考虑...
      </h3>
      <br>
      <NuxtLink class="w-full" to="/">
        <Button>回到主页</Button>
      </NuxtLink>
    </div>
  </div>
</template>
