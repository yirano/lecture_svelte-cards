<script>
    import { onMount } from 'svelte';
    import PostForm from '../components/PostForm.svelte'

    const apiBaseUrl = 'https://ndb99xkpdk.execute-api.eu-west-2.amazonaws.com/dev'
    let posts = []
    let editingPost = {
        body: '',
        title: '',
        id: null
    }
    let postLimit = 6;

    onMount(async()=> {
        const res = await fetch(apiBaseUrl + '/posts');
        posts = await res.json();
    })

    const editPost = (e, post) => {
        e.preventDefault()
        editingPost = post
    }

    const deletePost = (e, id) => {
        e.preventDefault()
        fetch(`${apiBaseUrl}/post/${id}`, {
            method: 'DELETE'
        })
        .then(res => {
            return res.json()
        })
        .then(() => {
            posts = posts.filter(p => p.id !== id)
        })
    }

    const addPost = (e) => {
        if(posts.find(p=>p.id===e.detail.id)) {
            const index = posts.findIndex(p=> p.id === e.detail.id)
            let postsUpdated = posts;
            postsUpdated.splice(index, 1, e.detail)
            posts = postsUpdated
            editingPost = {
                body: '',
                title: '',
                id: null
            }
        } else {
            posts = [e.detail, ...posts]

        }
    }

    const setLimit = () => {
        fetch(`${apiBaseUrl}/posts/${postLimit}`)
        .then(res => {
            return res.json()
        })
        .then(postsData => {
            posts = postsData
        })
    }
</script>

<style>
    .delete-btn {
        color: red !important;
    }
    .card .card-content .card-title {
        margin-bottom: 0;
    }
    .card .card-content p.timestamp {
        color: #999 !important;
        margin-bottom: 10px;
    }
</style>

<div class="row">
    <div class="col s6">
        <PostForm on:postcreated={addPost} {editingPost}/>
    </div>
    <div class="col s3" style="margin: 50px">
        <p>Limit number of posts</p>
        <input type="number"  bind:value={postLimit} />
        <button on:click={setLimit} class="waves-effect waves-light btn">
            Set
        </button>
    </div>
    {#if posts.length === 0}
        <h3>Loading posts...</h3>
    {:else}
        {#each posts as post}
            <div class="col s6">
                <div class="card">
                    <div class="card-content">
                        <p class="card-title">{post.title}</p>
                        <p class="timestamp">{post.createdAt}</p>
                        <p>{post.body}</p>
                    </div>
                    <div class="card-action">
                        <a href="/" class="edit-btn" on:click={(e) => editPost(e, post)}>Edit</a>
                        <a href="/" class="delete-btn" on:click={(e) => deletePost(e, post.id)}>Delete</a>
                    </div>
                </div>
            </div>
        {/each}
    {/if}


</div>