<script setup lang="ts">
import { ref, watch } from 'vue'
import { useIntersectionObserver, useWindowScroll } from '@vueuse/core'

import { useMainStore } from '@/stores/main.ts'
import { usePokemonStore } from '@/stores/pokemon.ts'

import SimpleCard from '@/components/SimpleCard.vue'
import NavBar from '@/components/NavBar.vue'
import InfoPage from '@/components/infoPage.vue'
import ScreenLoader from '@/components/screenLoader.vue'

const mainStore = useMainStore()
const pokemonStore = usePokemonStore()

const endPageTarget = ref(null)
const endPageTargettIsVisibleIsVisible = ref(false)

const currentCardGroup = ref(0)
const endPageVisibility = ref(true)

const pokemonSpriteBaseUrl = 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/'
const { y } = useWindowScroll({ behavior: 'smooth' })

function setGroup(index: number) {
  const groupValidation = [
    index <= 25,
    index > 25 && index <= 50,
    index > 50 && index <= 75,
    index > 75 && index <= 100,
    index > 100 && index <= 125,
    index > 125 && index <= 150,
    index > 150,
  ]
  return `${groupValidation.indexOf(true)}`
}
function setRef(index: number) {
  const refValidation = [
    index == 25,
    index == 50,
    index == 75,
    index == 100,
    index == 125,
    index == 150,
    index == 150,
  ]
  return `${refValidation.indexOf(true) + 1}`
}
function setInitPagination() {
  for(let index = 0; index < 7; index++) {
    const groupArray = document.getElementsByName(`group-${index}`)

    groupArray.forEach(
      (item) => index > 0 ? item.classList.add('hidden') : item.classList.remove('hidden')
    )
  }
}
function revealSection(index: number) {
  const groupArray = document.getElementsByName(`group-${index}`)

  groupArray.forEach((item) => {
    item.classList.remove('hidden')
    item.classList.add('show-card')
  })
  mainStore.setPaginationLoader(false)
  endPageVisibility.value = true
}
function handleScrollTopButton(show: boolean = false) {
  const scrollTopButton = document.getElementById('scroll-top-button')

  if (show) {
    scrollTopButton?.classList.remove('hidde')
    scrollTopButton?.classList.remove('transition-opacity')
    scrollTopButton?.classList.remove('duration-300')
    scrollTopButton?.classList.remove('transform')
    scrollTopButton?.classList.remove('translate-y-2')
    scrollTopButton?.classList.remove('opacity-0')
  } else {
    scrollTopButton?.classList.add('hidde')
    scrollTopButton?.classList.add('transition-opacity')
    scrollTopButton?.classList.add('duration-300')
    scrollTopButton?.classList.add('transform')
    scrollTopButton?.classList.add('translate-y-2')
    scrollTopButton?.classList.add('opacity-0')
  }
}

watch(endPageTargettIsVisibleIsVisible, ()=> {
  if(endPageTargettIsVisibleIsVisible.value) {
    mainStore.setPaginationLoader(true)
    endPageVisibility.value = false
    setTimeout(() => revealSection(currentCardGroup.value), 500)
    currentCardGroup.value ++
  }
})
watch(y, (scrollValue)=> {
  console.log(scrollValue)
  if (scrollValue > 100) {
    handleScrollTopButton(true)
  } else {
    handleScrollTopButton(false)
  }
})

pokemonStore.getPokemonList('kanto')
setTimeout(() => setInitPagination(), 500)
useIntersectionObserver(
  endPageTarget,
  ([entry], observerElement) => {
    endPageTargettIsVisibleIsVisible.value = entry?.isIntersecting || false
  },
)
</script>

<template>
  <NavBar
    activePage="home"
    :pokemon-count="pokemonStore.myPokemonTeam.length"
  />
  <ScreenLoader :show="mainStore.pokemonListLoader" />
  <InfoPage v-if="mainStore.homePokemonListNotFound">
    <template v-slot:content>
      <img
        class="
          mt-auto
          mx-auto
          w-[150px]
        "
        src="@/assets/icons/pikachu.svg"
        alt="pokeball"
      />
      <p
        class="
          mx-auto
          text-center
        "
      >
        Something went wrong, Team Rocket has attacked!
      </p>
      <small
        class="
          mx-auto
          mb-auto
          text-center
        "
      >
        (PokeAPI is not responding)
      </small>
    </template>
  </InfoPage>
  <div
    v-if="
      pokemonStore.pokemonList.pokemon_entries[0].entry_number &&
      !mainStore.homePokemonListNotFound
    "
    class="
      flex
      flex-wrap
      m-5
      bg-slate-100
    "
  >
    <template
      v-for="(pokemon, index) in pokemonStore.pokemonList.pokemon_entries"
      :key="index"
    >
      <div
        class="
          flex
          flex-col
          p-2
          w-full
          sm:w-6/12
          md:w-4/12
          2xl:w-2/12
        "
        :ref="`targetGroup${setRef(pokemon.entry_number)}`"
        :id="`entry-${pokemon.entry_number}`"
        :name="`group-${setGroup(pokemon.entry_number)}`"
      >
        <SimpleCard
          buttonText="Add to team"
          button-text-disabled="Your team is already full!"
          :button-action="() => { pokemonStore.addPokemonToTeam(pokemon.entry_number) }"
          :image-url="`${pokemonSpriteBaseUrl}${pokemon.entry_number}.png`"
          :title="`#${pokemon.entry_number} ${pokemon.pokemon_species.name}`"
          :disabled="pokemonStore.myPokemonTeam.length === 6"
        />
      </div>
    </template>
    <div
      v-if="mainStore.paginationLoader"
      class="
        flex
        w-full
        sm:w-6/12
        md:w-4/12
        2xl:w-2/12
      "
    >
      <img
        class="w-1/2 m-auto"
        src="@/assets/icons/loader.svg"
        alt="loader"
      >
    </div>
    <div
      v-if="endPageVisibility && currentCardGroup <= 5"
      ref="endPageTarget"
    />
  </div>
  <section
    id="scroll-top-button"
    class="
      rounded-full
      fixed
      bottom-4
      right-4
      z-10
      hidde
      transition-opacity
      duration-300
      transform
      translate-y-2
      opacity-0
    "
  >
    <button @click="y = 0">
      <img
        class="w-20"
        src="@/assets/icons/caret.svg"
        alt="to top"
      >
    </button>
  </section>
</template>