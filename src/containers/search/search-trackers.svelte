<script lang="ts">
  import SearchBar from "../../components/search-bar/search-bar.svelte";
  import type TrackerConfig from "../../modules/tracker/tracker";
  import { Interact } from "../../store/interact";
  import { Lang } from "../../store/lang";

  import { SearchStore, SearchTerm } from "../../store/search-store";
  import { TrackerStore } from "../../store/tracker-store";
  import TrackerList from "../board/trackers.svelte";
  export let term: string;

  let trackers: Array<TrackerConfig> = [];

  function change(evt: CustomEvent) {
    term = evt.detail;
    if ((evt.detail || "").length) {
      trackers = Object.keys($TrackerStore.trackers)
        .map((tag: string) => {
          return $TrackerStore.trackers[tag];
        })
        .filter((tracker: TrackerConfig) => {
          return tracker.label.search(term.toLowerCase()) > -1 || tracker.tag.search(term.toLowerCase()) > -1;
        });
      trackers = trackers;
    } else {
    }
  }

  function clear() {
    SearchStore.clear();
    term = undefined;
  }

  function close() {
    clear();
    SearchStore.close();
  }

  function more(evt: CustomEvent) {
    const tracker: TrackerConfig = evt.detail;
    Interact.elementOptions(tracker.getTrackableElement());
    close();
  }
  async function trackerTap(evt: CustomEvent) {
    const tracker: TrackerConfig = evt.detail;
    SearchStore.save(
      new SearchTerm({
        term: `#${tracker.tag}`,
        type: "trackers",
      })
    );
    Interact.trackerTap(tracker, $TrackerStore.trackers);
    close();
  }
</script>

<section class="n-panel stiff pt-2 mb-2">
  <SearchBar
    autofocus
    compact
    className="filler"
    searchTerm={term || ''}
    on:clear={clear}
    placeholder={Lang.t('search.search-trackers', 'Search Trackers...')}
    on:change={change} />
</section>
{#if term}
  <section class="search-results trackers n-panel scroll-y column">
    {#if trackers.length}
      <TrackerList view="list" {trackers} on:tap={trackerTap} on:more={more} hideAdd />
    {/if}
  </section>
{/if}
