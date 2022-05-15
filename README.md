# MyApp

Trying to debug when clicking the cancel_upload button
```elixir
** (MatchError) no match of right hand side value: nil
    (phoenix_live_view 0.17.9) lib/phoenix_live_view/upload.ex:73: Phoenix.LiveView.Upload.cancel_upload/3
```

Starting at `lib/my_app_web/live/post_live/index.ex:13` we enable uploads

```elixir
  def handle_params(params, _url, socket) do
    {:noreply,
     socket
     |> apply_action(socket.assigns.live_action, params)
     |> allow_upload(:image, accept: ~w(.jpg .jpeg), max_entries: 1)}
  end
```
And also pass it along to our form-component
```elixir
<%= if @live_action in [:new, :edit] do %>
  <.modal return_to={Routes.post_index_path(@socket, :index)}>
    <.live_component
      ...
      uploads={@uploads}
    />
  </.modal>
<% end %>
```

from there we can not navigate to `http://localhost:4000/posts/new` and click choose file and select a upload. After selecting a file this triggers `lib/my_app_web/live/post_live/form_component.html.heex:22` to render our entries block, making our cancel_upload button visible `lib/my_app_web/live/post_live/form_component.html.heex:35`

Clicking the cancel_upload fires off our `lib/my_app_web/live/post_live/index.ex:45` 
```elixir
  def handle_event("cancel-upload", %{"ref" => ref, "value" => value}, socket) do
    IO.inspect(
      ref: ref,
      value: value,
      socket: socket
    )

    {:noreply, socket |> cancel_upload(:image, ref)}
  end
```

We can see from our IO.inspect that our socket > assigns > uploads > image has 0 entries before calling 
cancel_upload/3

Inside `Phoenix.LiveView.Upload:71` `cancel_upload/3` we get the upload config from socket.assigns[:uploads][:image] then we call `UploadConfig.get_entry_by_ref/2` passing along our config and our ref which happends to be `0` from our console log IO.inspect. 

Looking at `phoenix_live_view/lib/phoenix_live_view/upload_config.ex:310` we see try to look in the config's entries which is empty per our console log inspect and thus returns nil 
```
    Enum.find(conf.entries, fn %UploadEntry{} = entry -> entry.ref === ref end)
```

with nil coming back to the response of `get_entry_by_ref/2` we see the error at `lib/phoenix_live_view/upload.ex:73: Phoenix.LiveView.Upload.cancel_upload/3` from nil not matching the left `%UploadEntry{}`.

Most of this code was based on the generators and from the documentations from `https://hexdocs.pm/phoenix_live_view/uploads.html`