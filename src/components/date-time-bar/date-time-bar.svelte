<script lang="ts">
  import dayjs from "dayjs";
  import type { Dayjs } from "dayjs";
  import { Lang } from "../../store/lang";
  import { onMount, createEventDispatcher } from "svelte";
  import { UserStore } from "../../store/user-store";
  import Calendar from "../calendar/calendar.svelte";
  import Text from "../text/text.svelte";
  import ListItem from "../list-item/list-item.svelte";
  import TimeSelect from "./time-select.svelte";

  const dispatch = createEventDispatcher();

  export let date: any = new Date().getTime(); // prop
  export let opened: boolean = false;
  export let style: string = "";

  let lastDate;
  let _date: Dayjs; // local
  let _opened: boolean = opened;

  $: if (date !== lastDate) {
    init();
  }

  function toggleOpen(): void {
    _opened = !_opened;
  }

  function init() {
    // Get provided date - default to today
    date = date || new Date().getTime();
    // SEt local date to maniuplate
    _date = date instanceof Date ? dayjs(date) : dayjs(new Date(date));
    // Set local opened
    _opened = opened;
    // Set last date to avoid uneeded reaction
    lastDate = date;
  }

  /**
   * Set the Date
   * Sets month, day, year - leaving time alone
   */
  function setDate(d: Dayjs) {
    _date = _date.set("month", d.get("month"));
    _date = _date.set("date", d.get("date"));
    _date = _date.set("year", d.get("year"));
    dispatch("change", _date);
    if (!opened && _opened) {
      _opened = false;
    }
  }

  onMount(init);
</script>

<style lang="scss">
  .date-time-bar {
    background-color: var(--color-solid);
    display: grid;
    flex-shrink: 1;
    flex-grow: 1;
    grid-template-columns: 1fr 0.7fr;
    align-items: center;

    button {
      height: 40px;
      flex-grow: 1;
      flex-shrink: 1;
      border: none;
      background-color: var(--color-solid);
      color: var(--color-primary-bright);
      margin: 0px;
      font-size: 0.7rem;
    }
    button.date {
      border-right: solid 1px var(--color-solid-2);
      text-align: left;
      padding-left: 10px;
    }
  }
  .view.visible {
    border-top: solid 1px var(--color-solid-2);
    margin-bottom: 8px;
  }
  :global(.date-time-bar-item.opened .left, .date-time-bar-item.opened .right) {
    display: none !important;
  }
</style>

{#if _date}
  <ListItem solo style="padding:0px; overflow:hidden; {style}" className="date-time-bar-item {_opened ? 'opened' : ''}">
    <div slot="left">
      <slot name="left" />
    </div>
    <div class="date-time-bar">
      <button
        class="date clickable"
        class:text-center={_opened}
        on:click={() => {
          toggleOpen();
        }}>
        <Text size="md" truncate>{_date.format('ddd MMM D YYYY')}</Text>
      </button>

      <TimeSelect
        is24Hour={$UserStore.meta.is24Hour ? true : false}
        bind:value={_date}
        on:change={(evt) => {
          setDate(evt.detail);
        }} />
    </div>
    <div slot="right">
      <slot name="right" />
    </div>
  </ListItem>
  <div class="animate up view date" class:visible={_opened} class:hidden={!_opened}>
    {#if _opened}
      <Calendar
        on:dayClick={(evt) => {
          setDate(evt.detail);
        }}
        initialDate={_date} />
    {/if}
  </div>
{/if}