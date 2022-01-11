import Callout from 'nextra-theme-docs/callout'

# Использование с Next.js

## Выборка данных на стороне клиента

Если ваша страница содержит часто обновляемые данные, и вам не нужно предварительно рендерить их, SWR идеально подходит и не требует специальной настройки: просто импортируйте `useSWR` и используйте хук внутри любых компонентов, которые используют данные.

Вот как это работает:

- Во-первых, сразу покажите страницу без данных. Вы можете отображать состояния загрузки для отсутствующих данных.
- Затем получите данные на стороне клиента и отобразите их по готовности.

Этот подход хорошо работает, например, для личных кабинетов. Поскольку личный кабинет является частной страницей, ориентированной на конкретного пользователя, поисковая оптимизация не имеет значения, и страницу не нужно предварительно рендерить. Данные часто обновляются, что требует выборки данных во время запроса.

## Предварительный рендеринг с данными по умолчанию

Если страницу необходимо предварительно отрендерить, Next.js поддерживает [2 формы пререндеринга](https://nextjs.org/docs/basic-features/data-fetching):  
**Статическая генерация (SSG)** и **Серверный рендеринг (SSR)**.

Вместе с SWR вы можете предварительно обработать страницу для SEO, а также иметь такие функции, как кэширование, ревалидация, отслеживание фокуса, интервальная повторная выборка на стороне клиента.

Вы можете использовать `fallback` опцию [`SWRConfig`](/docs/global-configuration), чтобы передать предварительно выбранные данные в качестве начального значения всех SWR хуков. Например, с `getStaticProps`:

```jsx
 export async function getStaticProps () {
  // `getStaticProps` выполняется на стороне сервера.
  const article = await getArticleFromAPI()
  return {
    props: {
      fallback: {
        '/api/article': article
      }
    }
  }
}

function Article() {
  // `data` всегда будет дуступна, так как находится в `fallback`.
  const { data } = useSWR('/api/article', fetcher)
  return <h1>{data.title}</h1>
}

export default function Page({ fallback }) {
  // SWR хуки в пределах `SWRConfig` будет использовать эти значения.
  return (
    <SWRConfig value={{ fallback }}>
      <Article />
    </SWRConfig>
  )
}
```

Страница всё ещё предварительно отрендерина. Она оптимизирована для SEO, быстро реагирует на запросы, а также полностью поддерживает SWR на стороне клиента. Данные могут быть динамическими и автоматически обновляться с течением времени.

<Callout emoji="💡">
  Компонент `Article` сначала отрендерит предварительно сгенерированные данные, а после гидратации страницы он снова получит последние данные, чтобы они были актуальными.
</Callout>

### Complex Keys

`useSWR` can be used with keys that are `array` and `function` types. Utilizing pre-fetched data with these kinds of keys requires serializing the `fallback` keys with `unstable_serialize`.

```jsx
import useSWR, { unstable_serialize } from 'swr'

export async function getStaticProps () {
  const article = await getArticleFromAPI(1)
  return {
    props: {
      fallback: {
        // unstable_serialize() array style key
        [unstable_serialize(['api', 'article', 1])]: article,
      }
    }
  }
}

function Article() {
  // using an array style key.
  const { data } = useSWR(['api', 'article', 1], fetcher)
  return <h1>{data.title}</h1>
}

export default function Page({ fallback }) {
  return (
    <SWRConfig value={{ fallback }}>
      <Article />
    </SWRConfig>
  )
}
```