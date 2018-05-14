<template>

    <div>

        <div v-if="action === 'add'">
            <h2>
                <span v-if="article.id">Edit "{{article.title}}"</span>
                <span v-else>Add New Article</span>
            </h2>
            <form @submit.prevent="addArticle" class="mb-3">

                <div class="form-group">
                    <input type="text" class="form-control" placeholder="Title" v-model="article.title" />
                </div>

                <div class="form-group">
                    <textarea type="text" class="form-control" placeholder="Body" v-model="article.body"></textarea>
                </div>

                <button type="submit" class="btn btn-light btn-block mb-2">Save</button>
                <button type="button" @click="fetchArticles()" class="btn btn-light btn-block">Back</button>

            </form>
        </div>

        <div v-else-if="action === 'show'">
            <div class="card card-body" v-if="article" v-bind:key="article.id">
                <h3>{{article.title}}</h3>
                <p>{{article.body}}</p>
                <hr />
                <button @click="editArticle(article)" class="btn btn-warning mb-2">Edit</button>
                <button @click="deleteArticle(article.id)" class="btn btn-danger">Delete</button>
            </div>
        </div>

        <div v-else="action === 'list'">
            <h2>Articles</h2>
            <div class="mb-2" v-if="action !== 'add'">
                <button @click="createArticle()" class="btn btn-success">Add New Article</button>
            </div>
            <nav aria-label="Page navigation example">
              <ul class="pagination">
                <li v-bind:class="[{disabled: !pagination.prev_page_url}]" class="page-item">
                    <a class="page-link" href="#" @click="fetchArticles(pagination.prev_page_url)">Previous</a>
                </li>
                <li class="page-item disabled">
                    <a class="page-link text-dark" href="#">
                        Page {{pagination.current_page}} of {{pagination.last_page}}
                    </a>
                </li>
                <li v-bind:class="[{disabled: !pagination.next_page_url}]" class="page-item">
                    <a class="page-link" href="#" @click="fetchArticles(pagination.next_page_url)">Next</a>
                </li>
              </ul>
            </nav>

            <div class="card card-body" v-for="article in articles" v-bind:key="article.id">
                <h3>{{article.title}}</h3>
                <p>{{article.body}}</p>
                <hr />
                <button @click="editArticle(article)" class="btn btn-warning mb-2">Edit</button>
                <button @click="deleteArticle(article.id)" class="btn btn-danger">Delete</button>
            </div>
        </div>
    </div>
</template>

<script>
export default{
    data() {
        return {
            articles:[],
            action: 'list',
            article: {
                id: '',
                title: '',
                bode: ''
            },
            article_id: '',
            pagination: {},
            edit: false
        }
    },

    created() {
        this.fetchArticles();
    },

    methods: {

        createArticle() {
            this.action = 'add';
        },

        showArticle(id) {
            this.action = 'show';
            fetch('api/article/'+id, {
                method: 'get'
            })
            .then(res => res.json())
            .then(data => {
                alert('Article removed');
                this.fetchArticles();
            })
            .catch(err => console.log(err));
        },

        fetchArticles(page_url) {
            this.action = 'list';
            this.article = {};
            this.edit = false;

            let vm = this;
            page_url = page_url || 'api/articles';
            fetch(page_url)
            .then(res => res.json())
            .then(res => {
                //console.log(res.data);
                this.articles = res.data;
                vm.makePagination(res.meta, res.links);
            })
            .catch(err => console.log(err));
        },

        makePagination(meta, links) {
            let pagination = {
                current_page: meta.current_page,
                last_page : meta.last_page,
                next_page_url: links.next,
                prev_page_url: links.prev
            };

            this.pagination = pagination;
        },


        deleteArticle(id) {
            if (confirm('Are you sure?')) {
                fetch('api/article/'+id, {
                    method: 'delete'
                })
                .then(res => res.json())
                .then(data => {
                    alert('Article removed');
                    this.fetchArticles();
                })
                .catch(err => console.log(err));
            }
        },

        addArticle() {
            if (this.edit === false) {
                fetch('api/article', {
                    method: 'post',
                    body: JSON.stringify(this.article),
                    headers: {
                        'content-type': 'application/json'
                    }
                })
                .then(res => res.json())
                .then(data => {
                    this.article.title = '';
                    this.article.body = '';
                    alert('Article added');
                    this.fetchArticles();
                })
                .catch(err => console.log(err));
            } else {
                // update
                fetch('api/article', {
                    method: 'put',
                    body: JSON.stringify(this.article),
                    headers: {
                        'content-type': 'application/json'
                    }
                })
                .then(res => res.json())
                .then(data => {
                    this.article.id = '';
                    this.article.title = '';
                    this.article.body = '';
                    alert('Article updated');
                    this.fetchArticles();
                })
                .catch(err => console.log(err));
            }
        },

        editArticle(article) {
            this.edit = true;
            this.action = 'add';
            this.article.id = article.id;
            this.article.article_id = article.id;
            this.article.title = article.title;
            this.article.body = article.body;
        }
    }
};
</script>