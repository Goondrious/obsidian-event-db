---
seriesFocus: the-show
episodeFocus: ep1-the-show
characterFocus: frodo-the-show
---

The following examples could be split up into various views, like a "Characters" page for any given show. You could explore it however you wanted manually or make it automated via templater/dataview (e.g. with yaml tags to define filters/groupings etc.).

The grouping/joining that's done in Notion requires `dataviewjs`.
# Queries

## List all series
```dataview
TABLE name, rating FROM "Series"
SORT rating desc
```


## List all Events from an Episode
```dataview
TABLE episode, location, type, rating FROM "Events" WHERE episode = [[]].episodeFocus
```


```dataviewjs
const EPISODE_FOLDER = '"Episodes"';

  

const CHARACTER_FOLDER = '"Characters"';

  

const EVENT_FOLDER = '"Events"';

  

const getCharactersForSeries = (series) => {

  const episodes = getEpisodesForSeries(series);

  

  const episodeCharacterIds = episodes.map(getCharacterIdsForEpisode);

  

  const characterIds = episodeCharacterIds.reduce((set, o) => {

    o.forEach((p) => set.add(p));

  

    return set;

  }, new Set());

  

  const characters = dv

  

    .pages(CHARACTER_FOLDER)

  

    .filter((o) => characterIds.has(o.id));

  

  return characters;

};

  

const getEventsForSeries = (series) => {

  const episodes = getEpisodesForSeries(series);

  

  const events = episodes.map((o) => {

    const events = getEventsForEpisodeId(o.id);

    console.log("events", events);

  

    events.forEach((p) => (p.episode = o));

  

    return events;

  });

  

  return events.flat();

};

  

const getEventsForCharacter = (character) => {

  const events = dv

    .pages(EVENT_FOLDER)

    .filter((o) => o.characters && o.characters.includes(character))

    .map((o) => {

      const episode = getEpisodeByEpisodeId(o.episode);

      o.episode = episode;

      return o;

    });

  

  console.log("character events", events, events.array());

  return events.array().flat();

};

  

const getEpisodeByEpisodeId = (episodeId) => {

  const episodes = dv.pages(EPISODE_FOLDER).filter((o) => o.id === episodeId);

  return episodes[0];

};

  

const getEpisodesForSeries = (series) => {

  const episodes = dv.pages(EPISODE_FOLDER).filter((o) => o.series === series);

  

  return episodes.array();

};

  

const getCharacterIdsForEpisode = (episode) => {

  const baseCharacters = episode.characters || [];

  

  const events = getEventsForEpisodeId(episode.id);

  

  const eventCharacters = events.map((o) => o.characters) || [];

  

  const characterIds = new Set([...baseCharacters, ...eventCharacters.flat()]);

  

  return characterIds;

};

  

const getEventsForEpisodeId = (episodeId) => {

  const events = dv.pages(EVENT_FOLDER).filter((o) => o.episode === episodeId);

  

  return events.array() || [];

};

  

const series = dv.current().seriesFocus;

const character = dv.current().characterFocus;

  

const charactersForSeries = getCharactersForSeries(series);

  

console.log("characters for series", charactersForSeries);

  

dv.header(2, "Characters in Series - " + series);

dv.table(

  ["name", "groups"],

  

  charactersForSeries.map((o) => [o.firstName + " " + o.lastName, o.groups])

);

  

const eventsForSeries = getEventsForSeries(series);

  

console.log("events for series", eventsForSeries);

dv.header(2, "Events in Series - " + series);

dv.table(

  ["Chronological Order", "Date Aired", "Name", "Location"],

  

  eventsForSeries

    .sort((o) => o.chronologicalOrder)

    .sort((o) => dv.date(o.episode.dateAired))

  

    .map((o) => [

      o.chronologicalOrder,

      o.episode.dateAired,

      o.file.name,

      o.location,

    ])

);

  

const eventsForCharacter = getEventsForCharacter(character);

console.log(eventsForCharacter);

dv.header(2, "Character Events - " + character);

dv.table(

  ["Chronological Order", "Date Aired", "Name", "Location"],

  

  eventsForCharacter

    .sort((o) => o.chronologicalOrder)

    .sort((o) => dv.date(o.episode.dateAired))

  

    .map((o) => [

      o.chronologicalOrder,

      o.episode.dateAired,

      o.file.name,

      o.location,

    ])

);
```

