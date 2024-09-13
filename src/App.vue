 <script setup lang="ts">
 import { ref, onMounted } from 'vue';
 import { watch } from 'vue';
 
 interface Repository {
   id: number;
   name: string;
   description: string;
   stars: number;
   forks: number;
   watchers: number;
 }
 
 const selectedLanguage = ref<string>('');
 const repositories = ref<Repository[]>([]);
 const isLoading = ref<boolean>(false);
 const hasError = ref<boolean>(false);
 
//  const languages = ref<{ [key: string]: string }>({});
 
 // Fonction pour tronquer la description à 20 mots
 const truncateDescription = (text: string, wordLimit: number) => {
   const words = text.split(' ');
   return words.length > wordLimit
     ? words.slice(0, wordLimit).join(' ') + '...'
     : text;
 };
 
 // Fetching the list of languages from the external JSON file
//  const fetchLanguages = async () => {
//    try {
//      const res = await fetch('https://raw.githubusercontent.com/kamranahmedse/githunt/master/src/components/filters/language-filter/languages.json');
//      languages.value = await res.json();
//    } catch (error) {
//      console.error("Erreur lors du chargement des langages", error);
//    }
//  };
interface Language {
  title: string;
  value: string;
}

const languages = ref<Language[]>([]);


 const fetchLanguages = async () => {
  try {
    const res = await fetch('https://raw.githubusercontent.com/kamranahmedse/githunt/master/src/components/filters/language-filter/languages.json');
    languages.value = await res.json() as Language[];
  } catch (error) {
    console.error("Erreur lors du chargement des langages", error);
  }
};

 
 // Fetching GitHub repositories based on the selected language
 const fetchRepositories = async () => {
   if (!selectedLanguage.value) return;
   isLoading.value = true;
   hasError.value = false;
   repositories.value = [];
 
   try {
     const res = await fetch(`https://api.github.com/search/repositories?q=language:${selectedLanguage.value}&sort=stars&order=desc&per_page=10&page=${Math.floor(Math.random() * 10) + 1}`);
     const data = await res.json();
     repositories.value = data.items.map((repo: any) => ({
       id: repo.id,
       name: repo.name,
       description: repo.description || 'Aucune description disponible.',
       stars: repo.stargazers_count,
       forks: repo.forks_count,
       watchers: repo.watchers_count
     }));
   } catch (error) {
     hasError.value = true;
   } finally {
     isLoading.value = false;
   }
 };
 
 onMounted(() => {
   fetchLanguages();
 });
 
 // Watcher to trigger the repository fetch when a language is selected
 watch(selectedLanguage, fetchRepositories);
 </script>
 
 <template>
   <div class="repository-finder">
     <div class="header">
       <div class="logo"></div>
       <h2>GitHub Repository Finder</h2>
     </div> 
     <!-- Language selector -->
      <select v-model="selectedLanguage" style="width: 40%; height: 50px; border-radius: 10px; border: solid black;">
        <option value="" disabled selected>Choisissez un langage</option>
        <option v-for="language in languages" :key="language.value" :value="language.value">{{ language.title }}</option>
      </select>

     <!-- Empty state if no language is selected -->
     <div v-if="!selectedLanguage" style="background-color: #e9ecef; width: 25%; margin: auto; padding: 40px; border-radius: 15px;">
       <p>Veuillez sélectionner un langage</p>
     </div>
 
     <!-- Loading state -->
     <div v-if="isLoading && selectedLanguage" style="background-color: #e9ecef; width: 25%; margin: auto; padding: 40px; border-radius: 15px;">
       <p>Chargement, veuillez patienter...</p>
     </div>
 
     <!-- Error state -->
     <div v-if="hasError && selectedLanguage" >
      <div  style="background-color: #ffc9c9; width: 25%; margin: auto; padding: 40px; border-radius: 15px;">
        <p>Erreur lors du chargement des dépôts GitHub</p>
      </div>
       <button @click="fetchRepositories" style="background-color: #e03131; margin-top:10px; width: 31%;">Réessayer</button>
     </div>
 
     <!-- Display repositories in grid -->
     <div v-if="repositories.length && selectedLanguage" class="repository-grid">
       <div v-for="repo in repositories" :key="repo.id" class="repository-card">
         <h2  style=" ">{{ repo.name }}</h2>
         <p>{{ truncateDescription(repo.description, 10) }}</p> 
         <div class="comdete">
            <p><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" style="fill: rgba(248, 255, 4, 1);"><path d="M21.947 9.179a1.001 1.001 0 0 0-.868-.676l-5.701-.453-2.467-5.461a.998.998 0 0 0-1.822-.001L8.622 8.05l-5.701.453a1 1 0 0 0-.619 1.713l4.213 4.107-1.49 6.452a1 1 0 0 0 1.53 1.057L12 18.202l5.445 3.63a1.001 1.001 0 0 0 1.517-1.106l-1.829-6.4 4.536-4.082c.297-.268.406-.686.278-1.065z"></path></svg> : {{ repo.stars }}</p>
            <p><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" style="fill: rgba(39, 39, 0, 1);"><path d="M5.559 8.855c.166 1.183.789 3.207 3.087 4.079C11 13.829 11 14.534 11 15v.163c-1.44.434-2.5 1.757-2.5 3.337 0 1.93 1.57 3.5 3.5 3.5s3.5-1.57 3.5-3.5c0-1.58-1.06-2.903-2.5-3.337V15c0-.466 0-1.171 2.354-2.065 2.298-.872 2.921-2.896 3.087-4.079C19.912 8.441 21 7.102 21 5.5 21 3.57 19.43 2 17.5 2S14 3.57 14 5.5c0 1.552 1.022 2.855 2.424 3.313-.146.735-.565 1.791-1.778 2.252-1.192.452-2.053.953-2.646 1.536-.593-.583-1.453-1.084-2.646-1.536-1.213-.461-1.633-1.517-1.778-2.252C8.978 8.355 10 7.052 10 5.5 10 3.57 8.43 2 6.5 2S3 3.57 3 5.5c0 1.602 1.088 2.941 2.559 3.355zM17.5 4c.827 0 1.5.673 1.5 1.5S18.327 7 17.5 7 16 6.327 16 5.5 16.673 4 17.5 4zm-4 14.5c0 .827-.673 1.5-1.5 1.5s-1.5-.673-1.5-1.5.673-1.5 1.5-1.5 1.5.673 1.5 1.5zM6.5 4C7.327 4 8 4.673 8 5.5S7.327 7 6.5 7 5 6.327 5 5.5 5.673 4 6.5 4z"></path></svg>:{{ repo.forks }}</p>
            <p><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" style="fill: rgba(39, 39, 0, 1);"><path d="M12 2C6.486 2 2 6.486 2 12s4.486 10 10 10 10-4.486 10-10S17.514 2 12 2zm0 18c-4.411 0-8-3.589-8-8s3.589-8 8-8 8 3.589 8 8-3.589 8-8 8z"></path><path d="M11 11h2v6h-2zm0-4h2v2h-2z"></path></svg>:{{ repo.watchers }}</p>
         </div>
       </div>
     </div>
   </div>
 </template>
 
 <style scoped>
 .repository-finder {
   width: 100%;
   margin: 0 auto;
   text-align: center;
 }
 
 select {
   width: 50%;
   padding: 8px;
   margin-bottom: 16px;
   color: black;
   background-color: white;
   font-size: 15px;
   cursor: pointer;
 }
 
 .repository-grid {
   display: grid;
   grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
   gap: 20px;
   margin-top: 20px;
   width:97%;
   margin:auto;
   margin-bottom:10px;
 }
 
 .repository-card {
   padding: 10px;
   border: 2px solid #000000;
   border-radius: 10px;
   background-color: #f9f9f9;
   text-align: left;
   display: flex;
  flex-direction: column;
 }
 
 .logo {
   background-color: black;
   width: 40px;
   height: 40px;
   border-radius: 15px;
 }
 
 .comdete{
  display:flex;
  gap:15px;
 }
 .header {
   display: flex;
   justify-content: center;
   align-items: center;
   width: 50%;
   margin: auto;
   gap:50px;
 }
 p{
  display: flex;
  align-items: center
 }
 svg{

  width: 17px;
  height: 17px;
 }
 </style>
 