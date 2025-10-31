<script setup lang="ts">
import { ref, computed, watch } from 'vue';

interface Carte {
  img: string;
  texte: string;
}

const cartesDisponibles: Carte[] = [
  { img: '/Asset/chat-noir.jpg', texte: 'Carte différente 1' },
  { img: '/Asset/paysage-crane.png', texte: 'Carte différente 2' },
  { img: '/Asset/paysage-citrouille.jpg', texte: 'Carte différente 3' },
  { img: '/Asset/squelette-fantome.jpg', texte: 'Carte différente 4' },
  { img: '/Asset/sombre-halloween.jpg', texte: 'Carte différente 5' },
  { img: '/Asset/dark-halloween.jpg', texte: 'Carte différente 6' },
  { img: '/Asset/halloween-sorciere.jpg', texte: 'Carte différente 7' },
];

const imageDos = '/Asset/fond-carte.jpg';
const nombreCartes = ref(6);

// 🧩 Génération des cartes (chaque carte est doublée et mélangée)
const cartesAffichees = computed(() => {
  const resultat: Carte[] = [];
  const nombrePaires = Math.floor(nombreCartes.value / 2);

  for (let i = 0; i < nombrePaires; i++) {
    const carte = cartesDisponibles[i % cartesDisponibles.length];
    resultat.push({ ...carte }, { ...carte }); // 2 copies distinctes
  }

  // Mélange
  for (let i = resultat.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [resultat[i], resultat[j]] = [resultat[j], resultat[i]];
  }

  return resultat;
});

// 🔁 États
const cartesRetournees = ref<boolean[]>(Array(nombreCartes.value).fill(false));
const cartesValidees = ref<boolean[]>(Array(nombreCartes.value).fill(false));
const cartesSelectionnees = ref<number[]>([]);

watch(nombreCartes, (nouveauNombre) => {
  cartesRetournees.value = Array(nouveauNombre).fill(false);
  cartesValidees.value = Array(nouveauNombre).fill(false);
  cartesSelectionnees.value = [];
});

// 🃏 Fonction pour retourner une carte
function retournerCarte(index: number) {
  // Ne rien faire si la carte est déjà validée ou déjà retournée
  if (cartesValidees.value[index] || cartesRetournees.value[index]) return;

  cartesRetournees.value[index] = true;
  cartesSelectionnees.value.push(index);

  // Si 2 cartes sont sélectionnées, on vérifie
  if (cartesSelectionnees.value.length === 2) {
    const [i1, i2] = cartesSelectionnees.value;
    const carte1 = cartesAffichees.value[i1];
    const carte2 = cartesAffichees.value[i2];

    if (carte1.img === carte2.img) {
      // ✅ Les deux cartes sont identiques → validées
      cartesValidees.value[i1] = true;
      cartesValidees.value[i2] = true;
      cartesSelectionnees.value = [];
    } else {
      // ❌ Pas identiques → retour après 1s
      setTimeout(() => {
        cartesRetournees.value[i1] = false;
        cartesRetournees.value[i2] = false;
        cartesSelectionnees.value = [];
      }, 1000);
    }
  }
}
</script>

<template>
  <div class="input-container">
    <label for="nombre">Nombre de cartes (pair, max 14) :</label>
    <input type="number" id="nombre" v-model.number="nombreCartes" min="2" max="14" step="2">
  </div>

  <div class="container">
    <div
      class="carte"
      v-for="(carte, index) in cartesAffichees"
      :key="index"
      @click="retournerCarte(index)"
    >
      <div
        class="carte-interne"
        :class="{
          retournee: cartesRetournees[index] || cartesValidees[index]
        }"
      >
        <div class="face-avant">
          <img :src="imageDos" alt="Dos de la carte" />
        </div>
        <div class="face-arriere">
          <img :src="carte.img" :alt="carte.texte" />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.input-container {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}

.container {
  display: grid;
  grid-template-columns: repeat(2, 160px);
  row-gap: 15px;
  justify-content: center;
}

/* Cartes carrées avec effet flip */
.carte {
  aspect-ratio: 1 / 1;
  perspective: 1000px;
}

.carte-interne {
  width: 100%;
  height: 100%;
  position: relative;
  transition: transform 0.5s;
  transform-style: preserve-3d;
}

.carte-interne.retournee {
  transform: rotateY(180deg);
}

.face-avant,
.face-arriere {
  position: absolute;
  inset: 0;
  backface-visibility: hidden;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 10px rgba(0,0,0,0.2);
}

.face-avant img,
.face-arriere img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.face-arriere {
  transform: rotateY(180deg);
}
</style scoped>
<style>
body {
  margin: 0;
  background-image: url('/Asset/fond-halloween.png'); /* 🔹 Mets ici le nom exact de ton image */
  background-size: cover;       /* L’image couvre tout l’écran */
  background-position: center;  /* Centrée */
  background-repeat: no-repeat; /* Pas de répétition */
  background-attachment: fixed; /* Effet "fixe" quand on scrolle */
}
</style>