<template>
  <div>
    welcome
  </div>
</template>

<script lang="ts" setup>
import {createClient, fetchExchange, gql, provideClient, ssrExchange as createSSRExchange, useQuery,} from '@urql/vue'
import {persistedExchange} from '@urql/exchange-persisted';

type SSRExchange = ReturnType<typeof createSSRExchange>
type SSRData = ReturnType<SSRExchange['extractData']>

let initialState = useState<SSRData | undefined>('urql');

const ssrExchange = createSSRExchange({
  isClient: process.client,
  initialState: initialState.value,
  staleWhileRevalidate: false,
})


const client = createClient({
  url: 'https://trygql.formidable.dev/graphql/apq-weather',
  exchanges: [
    persistedExchange({
      preferGetForPersistedQueries: true,
    }),
    ssrExchange,
    fetchExchange,
  ],
});

provideClient(client)

const random = (Math.random() + 1).toString(36).substring(7);

const LOCATIONS_QUERY = gql`
  query Locations($query: String!) {
    data_${random}: locations(query: $query) {
      id
      name
    }
  }
`;

const response = useQuery({
  query: LOCATIONS_QUERY,
  variables: {query: 'LON'},
});

const {data, fetching, error} = response;


initialState.value = ssrExchange.extractData();
</script>
