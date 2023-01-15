>路由配置*号报错
>
>caught Error: Catch all routes ("*") must now be defined using a param with a custom regexp.
>
>```js
>{
>  path: "/:catchAll(.*)",
>  name: "404",
>  component: NotFound
>},