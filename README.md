<h1 align="center">
  <img src="https://about.xkcd.uk/assets/xkcd.png">
  <br />
  xkcd.uk
</h1>

<p align="justify">
Hi, this is the about page for xkcd.uk, a simple xkcd URL shortener / lengthener running off CF workers. 
When inputting a number, you will be redirected to <code>xkcd.com/<em>number</em></code>. 
Otherwise, xkcd.uk conducts a search on DuckDuckGo: <code>\site:xkcd.com <em>string</em></code>, and follows the redirect, leading you to (hopefully) that comic.
</p>

```ts
import { load } from "cheerio";

const ddg_parsing_regex = /.*\; url=(.*)/
const xkcd_number_check = /^\/(\d+)$/

export default {
    async fetch(request, env, ctx) {
        const url = new URL(request.url);

        if (xkcd_number_check.test(url.pathname)) 
            // Hey, they're using a number!
            // Let's redirect them to xkcd.com/$1.
            return Response.redirect(`https://xkcd.com/${xkcd_number_check.exec(url.pathname)?.[1]}`)
        // Oh, they're not using a number...
        // Let's conduct a search, then.
        else {
            let response = await fetch("https://duckduckgo.com/?q=" + encodeURIComponent(`\\site:xkcd.com ${url.pathname.slice(1)}`));
            let html = await response.text()
            let $ = load(html)

            let content = $(`meta[http-equiv=refresh]`).get(0)?.attribs.content
            let ddg_redirect_pathname = content?.match(ddg_parsing_regex)?.[1]
            let ddg_url = new URL(`https://duckduckgo.com${ddg_redirect_pathname}`).searchParams.get("uddg")
            
            if (content && ddg_redirect_pathname && ddg_url) {
                return Response.redirect(ddg_url)
            }
            else return new Response(html, {headers: response.headers})
        }
    },
};
```

<p align="center">
  <em>^ this is the worker ^</em>
</p>
For examples, try <a href="https://xkcd.uk/standards">xkcd.uk/standards</a>, or <a href="https://xkcd.uk/927">xkcd.uk/927</a>.
