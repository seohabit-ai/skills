# SEOHabit skill

Use [SEOHabit](https://seohabit.com) with your coding agent to check public pages, work on specific SEO findings in your own project, and recheck the published result.

SEOHabit helps make your website easier for Google to find and understand, so people looking for what you offer can find you. This skill connects that workflow to your local development environment. Project source stays with you.

## Install

```sh
npx skills add seohabit-ai/skills --skill seohabit
```

Choose the coding agent and installation location in the installer. The package contains instructions and an API reference; it does not install a background service.

## Connect your website

1. [Sign in to SEOHabit](https://seohabit.com/login), then add or select your website.
2. Open [API keys](https://seohabit.com/app/account/keys) for that website. Create a project key with `project:read`, `checks:write`, and `tasks:write` for the full workflow. A key with only `project:read` is enough to read existing reports.
3. Make the key available to your coding agent through the local environment variable `SEOHABIT_API_KEY` or your existing local secret manager. Keep it out of chat, source files, URLs, screenshots and logs.

Free projects can run homepage audits within their current allowance. Saving improvement tasks and rechecking them require verified website ownership and an active site subscription. Installing this skill does not change your plan or buy a subscription.

## Use it

Try a request such as:

> Use SEOHabit to read my latest website report. Explain the findings and their evidence before making changes.

Or, when you want the agent to work on your project:

> Use SEOHabit to check my website and fix the missing page titles in this project. Keep unrelated changes, run the relevant checks, and follow this project's release rules. Recheck the task after the changes are published.

The agent reads the selected report, keeps its original finding indexes, and obtains source-free improvement instructions. It locates and changes files in your own environment. After publication, a fresh check can return `verified`, `needs_work`, or `unknown`. Preparing code, completing an audit job, and confirming a published improvement are separate results.

The instructions are written in English and ask the agent to explain results in your language. Raw website evidence, URLs, API fields and machine statuses retain their original form.

## Browser alternative

Compatible browser agents can use the WebMCP tools registered in your signed-in SEOHabit workspace. Other browsers retain copyable improvement instructions. Browser support is experimental; installing a skill does not add WebMCP support to an agent.

See the [agent documentation](https://seohabit.com/docs/agents) and [OpenAPI specification](https://seohabit.com/openapi.json) for current interfaces. Report skill problems through this repository's issues, or contact [SEOHabit support](https://seohabit.com/contact).

## License

MIT. The license covers this skill package and its documentation.
