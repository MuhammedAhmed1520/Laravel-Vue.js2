<template>
  <div>
    <h1>Articles</h1>
    <form class="mb-3" @submit.prevent="addArticle">
        <div class="form-group">
            <input type="text" class="form-control" placeholder="Title" v-model="article.title">
        </div>
        <div class="form-group">
            <textarea class="form-control" placeholder="Body" v-model="article.body"></textarea>
        </div>
        <button type="submit" class="btn btn-light btn-block">Save</button>

    </form>
    <nav aria-label="Page navigation example">
  <ul class="pagination">
    <li class="page-item" v-bind:class="[{disabled : !pagination.prev_page_url }]"
    @click="fetchArticles(pagination.prev_page_url)"><a class="page-link" href="#">Previous</a></li>
    <li class="page-item disabled"><a class="page-link" href="#">{{pagination.current_page}} of {{pagination.last_page}}</a></li>
    <li class="page-item" v-bind:class="[{disabled : !pagination.next_page_url }]"
    @click="fetchArticles(pagination.next_page_url)"><a class="page-link" href="#">Next</a></li>
  </ul>
</nav>
    <div class="card card-body mb-2" v-for="article in articles" v-bind:key="article.id">
        <h3>
          {{article.title}}
        </h3>
        <p>{{article.body}}</p>
        <hr>
        <button class="btn btn-primary mb-2" @click="editArticle(article)">Edit</button>
        <button class="btn btn-danger" @click="deleteArticle(article.id)">Delete</button>

    </div>
    </div>
</template>
<script>
    export default{
        data(){
            return {
                articles:[],
                article:{
                    id:'',
                    title:'',
                    body:''
                },
                article_id:'',
                pagination:{},
                edit:false
                  }
            },
            created(){
                this.fetchArticles();
            },
            methods:{
                fetchArticles(page_url){
                  let vm = this;
                  page_url = page_url || 'api/articles';
                    fetch(page_url)
                        .then(res => res.json())
                        .then(res => {
                            this.articles = res.data;
                            vm.makePagination(res.meta,res.links)
                          console.log(res.data)
                      }).catch(err => console.log(err))
                },
                makePagination(meta,links){
                    let pagination = {
                      current_page:meta.current_page,
                      last_page:meta.last_page,
                      next_page_url:links.next,
                      prev_page_url:links.prev
                    }
                    this.pagination = pagination;
                },
                deleteArticle(id){
                  // Use sweetalret2
                  this.$swal(
                    {
                       title: 'Are you sure?',
                       text: "You won't be able to revert this!",
                       type: 'warning',
                       showCancelButton: true,
                       confirmButtonColor: '#3085d6',
                       cancelButtonColor: '#d33',
                       confirmButtonText: 'Yes, delete it!'
                     }).then((result) => {
                       if (result.value) {
                         fetch('api/article/'+id,{
                           method:'delete'
                         })
                         .then(res => res.json())
                         .then(data => {
                           this.$swal(
                            'Deleted!',
                            'Your Article has been deleted.',
                            'success'
                            )
                           this.fetchArticles();
                         })

                         }
                       });



                },
                addArticle(){
                  if(!this.edit){
                    fetch('api/article',{
                      method:'post',
                      body:JSON.stringify(this.article),
                      headers:{
                        'content-type':'application/json'
                      }
                    })
                    .then(res => res.json())
                    .then(data => {
                      this.article.title = '',
                      this.article.body = '',
                      this.fetchArticles();
                      this.$swal(
                       'Added!',
                       'Your Article has been Added',
                       'success'
                       )
                    })
                  }else{
                    fetch('api/article',{
                      method:'put',
                      body:JSON.stringify(this.article),
                      headers:{
                        'content-type':'application/json'
                      }
                    })
                    .then(res => res.json())
                    .then(data => {
                      this.article.title = '',
                      this.article.body = '',
                      this.fetchArticles();
                      this.$swal(
                       'Updated!',
                       'Your Article has been Updated.',
                       'success'
                       )
                    })
                  }
                },
                editArticle(article){
                  this.edit = true;
                  this.article.id = article.id;
                  this.article.article_id = article.id;
                  this.article.title = article.title;
                  this.article.body = article.body;

                },
            }
        }

</script>
