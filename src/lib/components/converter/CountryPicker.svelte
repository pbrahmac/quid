<script lang="ts">
  import { buttonVariants } from "$lib/components/ui/button/index.js";
  import * as Dialog from "$lib/components/ui/dialog/index.js";
  import * as Drawer from "$lib/components/ui/drawer/index.js";
  import { Input } from "$lib/components/ui/input/index.js";
  import { ScrollArea } from "$lib/components/ui/scroll-area/index";
  import Countries from "$lib/util/countries.json";
  import {
    cn,
    fetchConversionData,
    formatCountryName,
    getEmoji,
    type Country,
  } from "$lib/utils";
  import Check from "lucide-svelte/icons/check";
  import ChevronsUpDown from "lucide-svelte/icons/chevrons-up-down";
  import { MediaQuery } from "runed";
  import { getContext } from "svelte";
  import Skeleton from "../ui/skeleton/skeleton.svelte";

  // types
  type CountriesList = {
    emoji: string;
    code: Country;
    name: string | undefined;
  }[];

  // props
  let {
    direction,
    currCode = $bindable(),
    otherCode,
  }: {
    direction: "from" | "to";
    currCode: Country;
    otherCode: Country;
  } = $props();

  // media query
  const isDesktop: MediaQuery = getContext("isDesktop");

  // state variables
  let open = $state(false);

  // full and filtered countries lists
  let countriesList: CountriesList = $state(
    Object.entries(Countries).map((country) => ({
      emoji: country[1],
      code: country[0] as Country,
      name: undefined,
    }))
  );
  let filteredCountriesList = $state(countriesList);

  // search and select handlers
  let searchValue = $state("");
  function handleSearch() {
    filteredCountriesList = countriesList.filter((country) =>
      Object.values(country)
        .map((field) => field && field.toLowerCase())
        .join(" ")
        .includes(searchValue.toLowerCase())
    );
  }
  function handleSelect(code: Country) {
    currCode = code;
    open = false;
  }

  // on component mount, fetch full country names for search
  $effect(() => {
    // fetch rates data
    fetchConversionData(otherCode).then((refreshedRates) => {
      // repopulate countries list with fetched data
      countriesList.splice(
        0,
        countriesList.length,
        ...Object.values(refreshedRates)
          .map((currency) => ({
            emoji: getEmoji(currency.code),
            code: currency.alphaCode as Country,
            name: formatCountryName(currency.name),
          }))
          .sort((a, b) => a.name.localeCompare(b.name))
      );
    });
  });
</script>

{#snippet countryPickerContent()}
  <Input bind:value={searchValue} oninput={() => handleSearch()} />
  <ul class="w-full" class:mt-4={!isDesktop.matches}>
    <ScrollArea
      class={cn(
        "sm:h-96",
        isDesktop.matches ? "h-[77svh] w-full" : "h-[40svh]"
      )}
    >
      <div class="rounded-md border w-full">
        {#each filteredCountriesList as country}
          <button
            class="flex w-full hover:bg-secondary items-center justify-between py-3 px-4 border-b last:border-b-0"
            onclick={() => handleSelect(country.code)}
          >
            <li class="w-full">
              <div class="flex items-baseline space-x-2">
                <p>{country.emoji}</p>
                <p>{country.code}</p>
                {#if country.name === undefined}
                  <Skeleton class="w-1/2 h-3" />
                {:else}
                  <span
                    class="text-muted-foreground text-sm text-nowrap max-w-64 text-ellipsis overflow-hidden"
                  >
                    ({country.name})
                  </span>
                {/if}
              </div>
            </li>
            {#if country.code === currCode}
              <Check />
            {/if}
          </button>
        {/each}
      </div>
    </ScrollArea>
  </ul>
{/snippet}

{#if isDesktop.matches}
  <Dialog.Root bind:open>
    <Dialog.Trigger class={cn(buttonVariants({ variant: "outline" }), "pr-3")}>
      <div class="flex items-center justify-between space-x-4 w-full">
        <p>{getEmoji(currCode)} {currCode}</p>
        <ChevronsUpDown />
      </div>
    </Dialog.Trigger>
    <Dialog.Content class="sm:max-w-[425px]">
      <Dialog.Header>
        <Dialog.Title>Convert {direction} Currency</Dialog.Title>
        <Dialog.Description>
          Select the currency code associated with your currency.
        </Dialog.Description>
      </Dialog.Header>
      {@render countryPickerContent()}
    </Dialog.Content>
  </Dialog.Root>
{:else}
  <Drawer.Root closeThreshold={0.9} bind:open>
    <Drawer.Trigger class={cn(buttonVariants({ variant: "outline" }), "pr-3")}>
      <div class="flex items-center justify-between space-x-4 w-full">
        <p>{getEmoji(currCode)} {currCode}</p>
        <ChevronsUpDown />
      </div>
    </Drawer.Trigger>
    <Drawer.Content class="outline-none p-4 h-[65svh]">
      <Drawer.Header class="text-left">
        <Drawer.Title>Convert {direction} Currency</Drawer.Title>
        <Drawer.Description>
          Select the currency code associated with your currency.
        </Drawer.Description>
      </Drawer.Header>
      {@render countryPickerContent()}
    </Drawer.Content>
  </Drawer.Root>
{/if}
