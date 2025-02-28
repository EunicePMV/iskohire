<script lang="ts">
    import { loadPosts, JobPosts, publishPost, unpublishPost } from "$lib/stores/post_store";
    import { loadApplicants, Applications } from "$lib/stores/application_store";
    import { onMount } from "svelte";
	import type { PostgrestError } from "@supabase/supabase-js";
    
    let isPostsLoading = true
    let isApplicationsLoading = false
    let postsError: PostgrestError | undefined  
    let applicationError: PostgrestError | undefined
    onMount(async () => {
        postsError = await loadPosts()
        JobPosts.subscribe(() => {
            isPostsLoading = false
        })
    })
    let toggle = true
    async function publish(id: number){
        toggle = true
        await publishPost(id)
        postsError = await loadPosts()
        JobPosts.subscribe(() => {
            toggle = false
        })
    }
    async function unpublish(id: number){
        toggle = true
        await unpublishPost(id)
        postsError = await loadPosts()
        JobPosts.subscribe(() => {
            toggle = false
        })
    }
    async function getApplicants(job_id: number | null){
        isApplicationsLoading = true
        applicationError = await loadApplicants(job_id)
        Applications.subscribe(() => {
            isApplicationsLoading = false
            $Applications
        })
    }
</script>

<style>
    .no-scrollbar::-webkit-scrollbar {
        display: none;
    }
</style>

<!-- scrolling and effect -->
<div class="flex">
    <div class="m-8">
        <div class="mb-5 w-[1450px] h-[344px] flex overflow-x-scroll">
        {#if isPostsLoading}
        {#each Array(5) as _, i} 
            <div style="opacity: {(100-i*20)/100}" class="bg-white card card-hover shadow-offset-x-0 shadow-offset-y-4 shadow-blur-4 shadow-spread-0 shadow-opacity-25 w-[277px] min-w-[277px] h-[309px] min-h-[309px] p-3 flex flex-col mr-3"></div>
        {/each}
        
        {:else if $JobPosts}
        {#each $JobPosts as post}  
        <div class="flex flex-col items-center">
            <form method="GET" action="emp-editjob">
                <input type="hidden" name="job_id" value={post.job_id}>
             <button class="flex p-1.5 bg-white color-black rounded-sm hover:rounded-3xl hover:bg-white transition-all duration-300 text-white">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" fill="none" viewBox="0 0 24 24" stroke="black" stroke-width="2">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
                    </svg>
                    </button> 
                <button on:click|preventDefault={() => getApplicants(post.job_id)} class="bg-white card card-hover shadow-offset-x-0 shadow-offset-y-4 shadow-blur-4 shadow-spread-0 shadow-opacity-25 w-[277px] min-w-[277px] h-[309px] min-h-[309px] p-3 flex flex-col mr-3">
                    <div class="flex flex-col"> 
                        <span class="text-black font-inter text-xl font-extrabold leading-normal text-center">
                            {post.job_title}
                        </span>
                        <span class="text-gray-600 font-inter text-sm font-normal leading-normal text-center">
                            {post.desc}
                        </span>
                    </div>
                    <div class="w-full mt-auto self-end">
                        <div class="grid grid-cols-2 gap-2 mt-auto mb-0">
                            <div class="flex justify-center">
                                <img src="./images/loc_type-employer.png" class="h-3.5 w-3.5 mt-auto mb-auto mr-2" alt="Location Type Logo">
                                <span class="text-gray-600 font-inter text-xs font-normal leading-normal mt-auto mb-auto">{post.loc_type}</span>
                            </div>
                            <div class="flex">
                                <img src="./images/emp_type-employer.png" class="h-3.5 w-3.5 mt-auto mb-auto mr-2" alt="Employment Type Logo">
                                <span class="text-gray-600 font-inter text-xs font-normal leading-normal mt-auto mb-auto">{post.employment_type}</span>
                            </div>
                            <div class="flex justify-center ">
                                <img src="./images/salary.png" class="h-4 w-4.5 mt-auto mb-5 ml-3" alt="Salary Logo">
                                <span class="text-gray-600 font-inter text-xs font-normal leading-normal mt-auto mb-5 ml-2">{post.salary}</span>
                            </div>
                            <div class="flex">
                            </div>
                        </div>
                        <div class="text-center">
                            {#key toggle}
                                {#if post.status =="PUBLISH"}
                                <button on:click|preventDefault={()=>publish(post.job_id)} class="bg-[#D2AC72] border border-solid border-[#AB7C7C] w-[100px] font-inter text-xs font-black text-white h-8 ml-auto mr-0 shadow-md variant-filled-tertiary cursor-pointer hover:bg-[#AD9673]">
                                    {post.status }
                                </button>
                                {:else if post.status =="PUBLISHED" || post.status == "UNPUBLISHED"}
                                <button on:click|preventDefault={()=>unpublish(post.job_id)} class="bg-[#417E1B] border border-solid border-[#AB7C7C] w-[100px] font-inter text-xs font-black text-white h-8 ml-auto mr-0 shadow-md variant-filled-tertiary cursor-pointer hover:bg-[#24470F]">
                                    {post.status }
                                </button>
                                {/if}
                            {/key}
                        </div>
                    </div>
                </button>

                </div>
                {/each}
            {:else if postsError}
                <div>{postsError.message}</div>
            {:else}
                <div>No Posts Yet!</div>
            {/if}
        </div>
        <!-- filter and applicants -->
        <div class="mb-5 w-screen w-max-screen flex">
            <!-- filter -->
            <div class="flex  flex-col w-[277px] h-[329px] bg-white shadow-md pt-5 pb-5 pl-3 pr-3 mr-2">
                <span class="text-black text-justify font-inter text-xl font-semibold leading-normal">
                    Filter
                </span>
                <div>
                    <form class="flex flex-col">
                        <select class="text-black card card-hover text-justify font-inter text-lg font-semibold leading-normal border border-gray-300 bg-white shadow-md mb-3">
                            <option selected>Field Experience</option>
                        </select>
                        <select class="text-black card card-hover text-justify font-inter text-lg font-semibold leading-normal border border-gray-300 bg-white shadow-md mb-3">
                            <option selected>Course</option>
                        </select>
                        <select class="text-black card card-hover text-justify font-inter text-lg font-semibold leading-normal border border-gray-300 bg-white shadow-md mb-3">
                            <option selected>Skills</option>
                        </select>
                    </form>
                    <div class="text-right">
                        <button class="variant-filled-tertiary cursor-pointer hover:bg-[#AD9673] border border-solid border-[#AB7C7C] w-[90px] text-white font-inter text-base font-extrabold leading-normal h-8 mt-5 shadow-md">
                            APPLY
                        </button>
                        <button class="bg-white cursor-pointer hover:bg-black hover:text-white border border-black border-opacity-20 w-[90px] text-[#702828] font-inter text-base font-extrabold leading-normal h-8 mt-5 shadow-md">
                            CLEAR
                        </button>
                    </div>
                </div>
            </div>
            <!-- applicants section -->
        <div class="bg-white shadow-md p-3 w-[1125px] max-w-[1125px]">
        {#if isApplicationsLoading}
            {#each Array(5) as _, i} 
                <div style="opacity: {(100-i*20)/100}" class="h-[60px] border border-gray-300 bg-white shadow-md w-[1125px] max-w-[1125px] flex-shrink-0 p-3 mb-5 card"></div>
            {/each}
        {:else if $Applications}
            {#each $Applications as application}
                <div class="border border-gray-300 bg-white shadow-md w-[1125px] max-w-[1125px] flex-shrink-0 p-3 mb-5 card">
                    <div class="flex">
                        <div class="flex flex-col w-8/12 mt-auto mb-auto">
                            <span class="text-black font-inter text-lg font-extrabold leading-normal">
                                {application.applicant_name}
                            </span>
                            <span class="text-gray-600 italic font-inter text-sm font-thin leading-normal">
                                {#if application.skill_titles != null}
                                    {#each application.skill_titles as skill}
                                        {#if skill !== null}
                                            {skill}, 
                                        {:else}
                                            Nothing to see here
                                        {/if}
                                        
                                    {/each}
                                {:else}
                                    Nothing to see here
                                {/if}
                            </span>
                        </div>
                        <div class="pr-3 border-r border-gray-500 w-2/12 mt-auto mb-auto">
                            <button class="bg-[#D2AC72] border border-solid border-[#AB7C7C] w-[134px] h-[38px] text-white font-inter text-base font-extrabold leading-normal h-8 shadow-md">
                                View Profile
                            </button>
                        </div>
                        <div class="border-r w-2/12 pl-3 flex justify-center mt-2">
                            <div class="transition-all duration-300 group cursor-pointer">
                                <img
                                src="/images/hire.png"
                                alt="save"
                                class="h-8 group-hover:hidden mr-3"
                            />
                            <img
                                src="/images/hired.png"
                                alt="save"
                                class="h-8 hidden group-hover:block mr-3"
                            />
                            </div>
                            
                            <div class="transition-all duration-300 group cursor-pointer">
                                <img
                                src="/images/wait.png"
                                alt="save"
                                class="h-8 group-hover:hidden mr-3"
                            />
                            <img
                                src="/images/waited.png"
                                alt="save"
                                class="h-8 hidden group-hover:block mr-3"
                            />
                            </div>
                            <div class="transition-all duration-300 group cursor-pointer">
                                <img
                                src="/images/delete.png"
                                alt="save"
                                class="h-8 group-hover:hidden mr-3"
                            />
                            <img
                                src="/images/deleted.png"
                                alt="save"
                                class="h-8 hidden group-hover:block mr-3"
                            />
                            </div>
                        </div>
                    </div>
                </div>
                
                {/each}
            {:else if applicationError}
            <div class="border border-gray-300 bg-white shadow-md w-[1125px] max-w-[1125px] h-[60px] flex-shrink-0 p-3 mb-5 card">
                <span class="text-black font-inter text-lg font-extrabold leading-normal">
                    <div>{applicationError.message}</div>
                </span>
            </div>
            {:else}
            <div class="border border-gray-300 bg-white shadow-md w-[1125px] max-w-[1125px] h-[60px] flex-shrink-0 p-3 mb-5 card">
                <span class="text-black font-inter text-lg font-extrabold leading-normal">
                    Nothing to see here!
                </span>
            </div>
            {/if}
            </div>
        </div>
    </div>
</div>