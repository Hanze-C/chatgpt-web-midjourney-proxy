<script setup lang="ts">
import { mlog, upImg } from '@/api';
import { runwayFeed, runwayFetch, runwayUpload } from '@/api/runway';
import { gptServerStore, homeStore } from '@/store';
import { useMessage,NInput,NButton, NTag,NSelect,NPopover,NSwitch } from 'naive-ui';
import { computed, onMounted, ref, watch } from 'vue';
import { SvgIcon } from '@/components/common';
import { t } from '@/locales'; 
import { RunwayTask } from '@/api/runwayStore';

const fsRef= ref() ;
const runway= ref<{image_prompt?:string,seed:number,text_prompt:string}>({image_prompt:'',seed:1675247627,text_prompt:''});
const st= ref({isDo:false,uploading:false, version:'gen2',time:5,image_as_end_frame:false});
const ms = useMessage();
const exRunway= ref<RunwayTask>()
async function  selectFile(input:any){
    mlog("selectFile", input.target.files[0])
    const file = input.target.files[0]  

    st.value.uploading= true
    try{
    let d= await runwayUpload( file,'DATASET_PREVIEW')
    mlog("runwayFetch",d)
    runway.value.image_prompt= d.url
    }catch(e :any){
       ms.error(e )
    }
     st.value.uploading= false
    
}
function getRandomInt(min: number, max: number): number {
    min = Math.ceil(min);
    max = Math.floor(max);
    return Math.floor(Math.random() * (max - min + 1)) + min;
}

const canPost = computed(() => {
    return (runway.value.image_prompt!='' || runway.value.text_prompt!='' ) && !st.value.isDo
})

const generate= async ()=>{
    st.value.isDo= true
    //runway.value.seed= getRandomInt(1675247627, 3275247627)
    let seed= getRandomInt(1375247627, 3975247627);
    try{
        let obj={
            "taskType": "gen2",
            "internal": false,
            "options": {
                "name": `Gen-2 ${seed}`,
                "seconds": 4,
                "gen2Options": {
                "mode": "gen2",
                "seed": seed,
                "interpolate": true,
                "upscale": false,
                "watermark": true,
                "motion_score": 22,
                "use_motion_score": true,
                "use_motion_vectors": false,
                "text_prompt":  runway.value.text_prompt,
                "image_prompt": runway.value.image_prompt, 
                "init_image": runway.value.image_prompt
                },
                "exploreMode": false,
                "assetGroupName": "Generative Video"
            },
           // "asTeamId": 17485144
        }

//         {
//   "name": "Gen-3 Alpha 2584627205, 笑起来, Cropped - cqkrcrc8j3",
//   "seconds": 5,
//   "text_prompt": "笑起来",
//   "seed": 2584627205,
//   "exploreMode": true,
//   "watermark": false,
//   "enhance_prompt": true,
//   "init_image": "https://d2jqrm6oza8nb6.cloudfront.net/previews/21fb66fc-c9d0-4c92-863d-623b77ab742b.webp?_jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJrZXlIYXNoIjoiNjI5MzQ4YTc0ODIwYWZiMiIsImJ1Y2tldCI6InJ1bndheS1kYXRhc2V0cyIsInN0YWdlIjoicHJvZCIsImV4cCI6MTcyMjY0MzIwMH0.x5f94vMk6Yt4dQTw4ueBnWOJ1EFRqOAp_vaLUcT5bs0",
//   "resolution": "720p",
//   "assetGroupName": "Generative Video"
// }
        let gen3= {
                "taskType": "europa",
                "internal": false,
                "options": {
                    "name": `Gen-3 Alpha  ${seed}`,
                    "seconds": st.value.time,
                    "text_prompt":runway.value.text_prompt,
                    "seed":seed,
                    "exploreMode": true,
                    "watermark": false,
                    "enhance_prompt": true,
                    "width": 1280,
                    "height": 768,
                    "image_as_end_frame": false,
                    "assetGroupName": "Generative Video",
                    "init_image": runway.value.image_prompt,
                    "resolution": '720p'// runway.value.image_prompt,
                    ,"extended_from_task_id":(exRunway.value&&exRunway.value.id)?exRunway.value.id:undefined
                    ,"init_video": ( exRunway.value && exRunway.value.artifacts && exRunway.value.artifacts[0].url)?exRunway.value.artifacts[0].url:undefined
                },
            //    "asTeamId": 17511575
        }
        let gen3_trubo=    {
                "taskType": "gen3a_turbo",
                "internal": false,
                "options": {
                    "name": `Gen-3 Alpha Turbo ${seed}`,
                    "seconds":st.value.time,
                    "text_prompt": runway.value.text_prompt ,
                    "seed": seed,
                    "exploreMode": false,
                    "watermark": false,
                    "enhance_prompt": true,
                    "init_image":  runway.value.image_prompt,
                    "resolution": "720p",
                    "image_as_end_frame": false,
                    "assetGroupName": "Generative Video"
                   ,"extended_from_task_id":(exRunway.value&&exRunway.value.id)?exRunway.value.id:undefined
                    ,"init_video": ( exRunway.value && exRunway.value.artifacts && exRunway.value.artifacts[0].url)?exRunway.value.artifacts[0].url:undefined
                }
        }
        let v_gen3={
            "taskType": "europa",
            "internal": false,
            "options": {
                "name": `Gen-3 Alpha  ${seed}`,
                "seconds": st.value.time,
                "text_prompt":runway.value.text_prompt,
                "seed":seed,
                "exploreMode": true,
                "watermark": false,
                "enhance_prompt": true,
                "video_prompt":  runway.value.image_prompt ,
                "structure_transformation": 0.3,
                "width": 1280,
                "height": 768,
                "assetGroupName": "Generative Video"
            }
        }
  

        if( obj.options.gen2Options.image_prompt==''){
            delete obj.options.gen2Options.image_prompt;
            delete obj.options.gen2Options.init_image;
        }
        if( gen3.options.init_image=='' ){
            delete gen3.options.init_image;
            delete gen3_trubo.options.init_image;
        }
        if( !gen3.options.init_video  ){
            delete gen3.options.init_video;
            delete gen3_trubo.options.init_video;
        }
        if( !gen3.options.extended_from_task_id  ){
            delete gen3.options.extended_from_task_id;
            delete gen3_trubo.options.extended_from_task_id;
        }
        gen3.options.image_as_end_frame=st.value.image_as_end_frame
        gen3_trubo.options.image_as_end_frame=st.value.image_as_end_frame
        
        gen3.options.exploreMode= st.value.version=='europa'
        v_gen3.options.exploreMode= st.value.version=='europa'
        let sobj:any = gen3;
        if(  st.value.version=='gen2' ){
            sobj= obj
        }
        if(  st.value.version=='gen3a_turbo' ){
            sobj= gen3_trubo
            if(gen3_trubo.options.init_image=='') {
                ms.error( t('video.gen3a_turbo_img') )
                return 
            }
        }
        if(runway.value.image_prompt && isMp4(runway.value.image_prompt)){
            if( st.value.version=='gen2'){
                ms.error( 'gen2 不支持视频' )
                return 
            }
            v_gen3.taskType='europa'
            if( st.value.version=='gen3a_turbo' ){
                v_gen3.taskType='gen3a_turbo'
            }
            sobj= v_gen3
        }
       // const d=  await runwayFetch('/tasks', st.value.version=='gen2'?obj: gen3 ) 
        const d=  await runwayFetch('/tasks',  sobj ) 
        mlog("runwayGen2",d) 
        d.task && d.task.id&& runwayFeed(d.task.id)
    }catch(e:any){
        ms.error(e)
    }
    st.value.isDo=false

}
const isMp4=(url:string)=>{
    return url.indexOf('.mp4')>0
}

const mvOption= [
{label: t('video.rwgen2'),value: 'gen2'}
,{label:t('video.rwgen3'),value: 'europa'}
,{label:t('video.rwgen3fast'),value: 'europa-fast'}
,{label:t('video.rwgen3turbo'),value: 'gen3a_turbo'}
 ]
 const timeOption= [
{label: 'Duration: 5s',value: 5}
,{label:'Duration: 10s',value: 10}
 ]

 

const clearInput=()=>{
    runway.value.image_prompt ='' 
    runway.value.text_prompt =''
    exRunway.value= undefined
}
watch(()=>st.value.version,(n:string)=>{
    gptServerStore.setMyData({RRUNWAY_VERSION:n})
})
onMounted(() => {
    homeStore.setMyData({ms:ms})
    st.value.version= gptServerStore.myData.RRUNWAY_VERSION?gptServerStore.myData.RRUNWAY_VERSION: 'gen2'
});

watch(()=>homeStore.myData.act, (n)=>{
     if(n=='runway.extend'){
       mlog("runway.extend", homeStore.myData.actData )
       exRunway.value = homeStore.myData.actData as RunwayTask
     }
});
</script>
<template>
<div > 
    <div  class="pt-1"  >
        <n-select v-model:value="st.version" :options="mvOption" size="small" />
    </div>
    <div class="pt-1">
      <n-input v-model:value="runway.text_prompt" 
                :placeholder="$t('video.descpls')"  type="textarea"  size="small"   
                :autosize="{ minRows: 3, maxRows: 12  }"  />
    </div>

    <div v-if="exRunway" class="pt-1">
        <div class="flex justify-between items-center">
            <div  >
                <n-popover trigger="hover">
                    <template #trigger>
                    <div class="line-clamp-1">
                    {{ $t('video.extend') }}: 
                     <template   v-if="exRunway.options.text_prompt">{{ exRunway.options.text_prompt }}</template>
                     <template v-else  >{{ exRunway.options.gen2Options?.text_prompt?exRunway.options.gen2Options.text_prompt: exRunway.name }}</template>
                    </div>
                    </template>
                    <div class=" max-w-[300px]">{{exRunway.id}}</div>
                    
                    <div v-if="exRunway.taskType=='gen3a'" >Version: Gen-3</div>
                    <div v-if="exRunway.taskType=='gen3a_turbo'" >Version: Gen-3-turbo</div>
                    <div v-if="exRunway.taskType=='gen2'" >Version: Gen-2</div>
                    <div v-if="exRunway.createdAt" >createdAt: {{ new Date( exRunway.createdAt).toLocaleString() }}</div>
                    <div class=" max-w-[300px]" v-if="exRunway.options.text_prompt">{{ exRunway.options.text_prompt }}</div>
                    <div class=" max-w-[300px]">{{ exRunway.options.gen2Options?.text_prompt?exRunway.options.gen2Options.text_prompt: exRunway.name }}</div>
                
                </n-popover>
            </div>
        </div>
        <div class="relative flex items-center justify-center bg-white bg-opacity-10 rounded-[5px] overflow-hidden aspect-[16/8.85] ">
            <video   loop  playsinline  controls v-if="exRunway.artifacts && exRunway.artifacts[0].url"
                referrerpolicy="no-referrer" :poster="exRunway.artifacts[0].previewUrls[0]" 
                class="w-full h-full object-cover"  >
                <source  :src="exRunway.artifacts[0].url" referrerpolicy="no-referrer" type="video/mp4"  >
            </video>   
        </div>
            
    </div>

    <div  class="pt-1" v-if="st.version!='gen2'" >
        <n-select v-model:value="st.time" :options="timeOption" size="small" />
    </div>



    <div class="pt-1">
        <div class="flex justify-between  items-end">
            <div class=" relative"> 
                <input type="file"  @change="selectFile"  ref="fsRef" style="display: none" accept="image/jpeg, image/jpg, image/png, image/gif, .mp4"/>
                
                <!-- <div v-if="st.version=='europa'"
                 class="h-[80px] w-[80px] overflow-hidden rounded-sm border border-gray-400/20 flex justify-center items-center"
                >
                 {{ $t('video.nosup') }} 
                </div> -->
                <div   class="h-[80px] w-[80px] overflow-hidden rounded-sm border border-gray-400/20 flex justify-center items-center cursor-pointer" @click=" fsRef.click()">
                  
                   
                    <SvgIcon icon="line-md:uploading-loop" class="text-[60px] text-green-300"  v-if="st.uploading"  ></SvgIcon>
                    <!-- <video :src="runway.image_prompt" v-else-if="runway.image_prompt && isMp4(runway.image_prompt)" /> -->
                    <video   loop  playsinline    v-else-if="runway.image_prompt && isMp4(runway.image_prompt)"
                        referrerpolicy="no-referrer" 
                        class="w-full h-full object-cover"  >
                        <source  :src="runway.image_prompt" referrerpolicy="no-referrer" type="video/mp4"  >
                    </video>  
                    <img :src="runway.image_prompt" v-else-if="runway.image_prompt" />
                    <div class="text-center"  v-else >{{ $t('video.selectimg') }}</div> 
                   
                </div>

                <div class=" absolute bottom-[-5px] right-[-15px]" v-if="runway.image_prompt&&st.version!='gen2' ">
                     <a :href="runway.image_prompt" class="cursor-pointer" target="_blank" v-if="isMp4(runway.image_prompt) ">
                      <NTag  type="success" size="small"  class="cursor-pointer" round :bordered="false">Vidoe</NTag>
                     </a>
                     <NSwitch v-model:value="st.image_as_end_frame" size="small" v-else>
                        <template #checked>尾帧</template> 
                        <template #unchecked>首帧</template>
                    </NSwitch>
                   
                </div>
            </div>
            <div class="text-right ">
                <div class="pb-1 flex justify-between items-center">
                    
                       
                     
                        <NTag v-if="runway.text_prompt!='' || runway.image_prompt!='' || exRunway" type="success" size="small" round  ><span class="cursor-pointer" @click="clearInput()" >{{$t('video.clear')}}</span></NTag>
                     
                </div>
                <NButton  :loading="st.isDo" type="primary" :disabled="!canPost" @click="generate()"><SvgIcon icon="ri:video-add-line"  /> {{$t('video.generate')}}</NButton> 
            </div>
        </div>
    </div>

    <div class="pt-2 text-[12px]" v-html="$t('video.runwayinfo')">
        
    </div>
</div>
</template>