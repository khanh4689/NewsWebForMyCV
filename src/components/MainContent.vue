<template>
    <div class="news-container">
        <header class="news-header">
            <h1 class="news-title">Tin Tức Mới Nhất</h1>
        </header>

        <nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-3">
            <div class="container">
                <a class="navbar-brand" href="#">Báo Mới</a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="collapse navbar-collapse" id="navbarNav">
                    <ul class="navbar-nav">
                        <li class="nav-item" v-for="category in categories" :key="category.id">
                            <a class="nav-link" :class="{ active: selectedCategory === category.id }" @click="filterNews(category.id)">
                                <i :class="category.icon"></i> {{ category.name }}
                            </a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>

        <div v-if="filteredPosts.length" class="container mb-4">
    <div class="featured-post mb-5">
        <img :src="filteredPosts[0].imageFile" class="w-100 rounded mb-3" style="height: 300px; object-fit: cover;">
        <h2 class="fw-bold">{{ filteredPosts[0].title }}</h2>
        <p class="text-muted">{{ filteredPosts[0].description }}</p>
        <router-link :to="`/news/${filteredPosts[0].id}`" class="btn btn-primary">Đọc tiếp</router-link>
    </div>
</div>
<div v-if="filteredPosts.length" class="container">
    <div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4">
        <div v-for="post in filteredPosts" :key="post.id" class="col">
            <div class="card news-card h-100">
                <img :src="post.imageFile" class="card-img-top" alt="News image" />
                <div class="card-body d-flex flex-column">
                    <h5 class="card-title">{{ post.title }}</h5>
                    <p class="card-text small text-muted mb-1">
                        <i class="bi bi-person"></i> <strong>Tác giả:</strong> {{ post.Author || 'Ẩn danh' }} |
                        <i class="bi bi-calendar"></i> <strong>Ngày đăng:</strong> {{ formatDate(post.DatePost) }}
                    </p>
                    <p class="card-text mb-2">{{ post.description || 'Không có mô tả.' }}</p>
                    <router-link :to="`/news/${post.id}`" class="btn btn-outline-primary mt-auto">Xem chi tiết</router-link>
                </div>
            </div>
        </div>
    </div>
</div>

</div>

</template>

<script>
import { collection, getDocs } from "firebase/firestore";
import { db } from "@/firebase";

export default {
    data() {
        return {
            selectedCategory: 'all',
            categories: [
                { id: 'all', name: 'Tất Cả', icon: 'bi bi-newspaper' },
                { id: 'Worlds', name: 'Thế Giới', icon: 'bi bi-globe' },
                { id: 'Technology', name: 'Công Nghệ', icon: 'bi bi-cpu' },
                { id: 'Sports', name: 'Thể Thao', icon: 'bi bi-trophy' },
                { id: 'Entertaiment', name: 'Giải Trí', icon: 'bi bi-film' }
            ],
            posts: []
        };
    },
    computed: {
        filteredPosts() {
            if (this.selectedCategory === 'all') return this.posts;
            return this.posts.filter(post => post.category === this.selectedCategory);
        }
    },
    async created() {
        try {
            const querySnapshot = await getDocs(collection(db, "News"));
            this.posts = querySnapshot.docs.map(doc => {
                let data = doc.data();
                if (data.DatePost && typeof data.DatePost.toDate === "function") {
                    data.DatePost = data.DatePost.toDate();
                }
                return { ...data, id: doc.id };
            });
        } catch (error) {
            console.error("Lỗi khi lấy tin tức:", error);
        }
    },
    methods: {
        filterNews(category) {
            console.log("Chọn danh mục:", category); // Kiểm tra dữ liệu category
            this.selectedCategory = category;
        },
        formatDate(date) {
            return date ? new Date(date).toLocaleDateString() : "Không xác định";
        }
    }
};
</script>

<style scoped>
.news-container {
    background-color: #f4f4f4;
    padding-bottom: 50px;
    min-height: 100vh;
}

.news-header {
    text-align: center;
    background: white;
    padding: 30px 20px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    margin-bottom: 20px;
}

.news-title {
    font-size: 2.5rem;
    font-weight: 700;
    color: #333;
}

.news-card {
    transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
    border: none;
    border-radius: 10px;
    overflow: hidden;
    background: white;
}

.news-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 6px 15px rgba(0, 0, 0, 0.1);
}

.card-title {
    font-size: 1.2rem;
    font-weight: 600;
    color: #222;
}

.card-text {
    font-size: 0.95rem;
    color: #555;
}

.navbar {
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.navbar-nav .nav-link {
    transition: 0.3s ease;
}

.navbar-nav .nav-link.active,
.navbar-nav .nav-link:hover {
    background: #0056b3;
    color: #fff !important;
    border-radius: 4px;
}

.btn-outline-primary {
    font-weight: 500;
    border-radius: 20px;
}

</style>
