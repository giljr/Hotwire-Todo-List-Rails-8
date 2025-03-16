## Build a Real-Time Todo List with Rails 8 and Turbo Streams

### Create Todos List Demo! #HotwirePoweredSeries — Episode 4


Welcome to the Todos List Demo! In this tutorial, you’ll learn how to create a Rails 8 app with Turbo Streams support, enabling real-time updates as you add, edit, or remove tasks.

If this is your first time in the series, consider starting with the introductory episode.

### Prerequisites

Before starting, ensure you have the following installed:

    Ruby (at least 3.2)

    Rails 8

Install Rails 8 (if not already installed):

    gem install rails --pre

#### 1. Create a New Rails 8 App

Generate a new Rails app called turbo_demo:
```bash
rails new turbo_demo
cd turbo_demo
code .
```
#### 2. Generate Models & Controllers

Create a Todo model and controller:
```bash
rails generate scaffold Todo title:string status:string
rails db:migrate
```
#### 3. Modify Routes

Edit `config/routes.rb`:
```ruby
Rails.application.routes.draw do
  resources :todos
  root "todos#index"
end
```
#### 4. Modify Model

Edit `app/models/todo.rb`:
```ruby
class Todo < ApplicationRecord
  after_create_commit -> { broadcast_append_to "todos", target: "todos-list" }
  after_update_commit -> { broadcast_replace_to "todos" }
  after_destroy_commit -> { broadcast_remove_to "todos" }
end
```
#### 5. Implement Turbo Stream Actions

Edit `app/controllers/todos_controller.rb`:
```ruby
class TodosController < ApplicationController
  before_action :set_todo, only: %i[show edit update destroy]

  def index
    @todos = Todo.all
  end

  def create
    @todo = Todo.new(todo_params)
    
    respond_to do |format|
      if @todo.save
        format.turbo_stream
        format.html { redirect_to todos_url, notice: "Todo was successfully created." }
      else
        format.html { render :new, status: :unprocessable_entity }
      end
    end
  end

  def update
    respond_to do |format|
      if @todo.update(todo_params)
        format.turbo_stream
        format.html { redirect_to todos_url, notice: "Todo updated successfully." }
      else
        format.html { render :edit }
      end
    end
  end

  def destroy
    @todo.destroy!
    respond_to do |format|
      format.turbo_stream
      format.html { redirect_to todos_url, notice: "Todo was successfully destroyed." }
    end
  end

  private

  def set_todo
    @todo = Todo.find(params[:id])
  end

  def todo_params
    params.require(:todo).permit(:title, :status)
  end
end
```
#### 6. Update Views with Turbo Frames & Streams

Modify `app/views/todos/index.html.erb`:
```html
<h1>Todos</h1>

<turbo-frame id="new_todo">
  <%= render "form", todo: Todo.new %>
</turbo-frame>

<%= turbo_stream_from "todos" %>
<ul id="todos-list">
  <%= render @todos %>
</ul>
```
#### 7. Modify _todo.html.erb

Edit `app/views/todos/_todo.html.erb`:
```html
<li id="<%= dom_id(todo) %>" class="todo-item">
  <span class="todo-status">
    <% if todo.status == 'complete' %>
      [x]
    <% else %>
      [ ]
    <% end %>
  </span>

  <span class="todo-title <%= 'completed' if todo.status == 'complete' %>">
    <%= todo.title %>
  </span>

  <div class="todo-actions">
    <% if todo.status == 'complete' %>
      <%= button_to "Incomplete", todo_path(todo, todo: { status: 'incomplete' }), method: :patch, data: { turbo: false }, class: "inline-button" %>
    <% else %>
      <%= button_to "Complete", todo_path(todo, todo: { status: 'complete' }), method: :patch, data: { turbo: false }, class: "inline-button" %>
    <% end %>
    
    <%= button_to "Delete", todo_path(todo), method: :delete, data: { turbo: false, confirm: "Are you sure?" }, class: "delete-button" %>
  </div>
</li>
```
#### 8. Create create.turbo_stream.erb

Edit `app/views/todos/create.turbo_stream.erb`:
```html
<%= turbo_stream.append "todos-list" do %>
  <%= render "todo", todo: @todo %>
<% end %>

<%= turbo_stream.replace "#{dom_id(Todo.new)}_form" do %>
  <%= render "form", todo: Todo.new %>
<% end %>
```
#### 9. Styling

Add this CSS to `app/assets/stylesheets/application.css`:
```html
 /* turbo-frame {
    display: block;
    border: 1px solid blue;
} */

.completed {
  text-decoration: line-through;
  color: #888; /* Optional: Change the color to indicate completion */
}

.todo-item {
    display: flex;
    align-items: center;
    gap: 16px; /* Space between elements */
    padding: 8px;
    border-bottom: 1px solid #eee; /* Optional: Add a separator between todos */
  }
  
  .todo-status {
    font-family: monospace;
    font-size: 1.2em;
  }
  
  .todo-title {
    flex: 1; /* Allow the title to take up remaining space */
    font-size: 1.2em;
  }
  
  .todo-title.completed {
    text-decoration: line-through;
    color: #888;
  }
  
  .todo-actions {
    display: flex;
    gap: 8px; /* Space between buttons */
  }
  
  .inline-button, .delete-button {
    padding: 4px 8px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.9em;
  }
  
  .inline-button {
    background-color: #91bbe7;
    color: white;
  }
  
  .inline-button:hover {
    background-color: ;
  }
  
  .delete-button {
    background-color: #e7858f;
    color: white;
  }
  
  .delete-button:hover {
    background-color: #e7858f;
  }
```
### Running the App

To start the Rails server:

    rails server

Visit http://localhost:3000 in your browser to see your real-time Todo List in action!

This tutorial is part of the #HotwirePoweredSeries. If you find this useful, check out my Medium and GitHub for more updates!

Happy coding! 🚀


## Authors

- [@jaythree](https://www.linkedin.com/in/giljrx/)


## Tutorial

- [Episode 4](https://medium.com/jungletronics/build-a-real-time-todo-list-with-rails-8-and-turbo-streams-e1cd6b9cee1d): **Build a Real-Time Todo List with Rails 8 and Turbo Streams** - Create Todos List Demo! #HotwirePoweredSeries


## License

[MIT](https://choosealicense.com/licenses/mit/)

