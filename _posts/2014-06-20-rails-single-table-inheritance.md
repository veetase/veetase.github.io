---
comments: true
date: 2014-06-20 18:28:11
layout: post
title: Rails Single Table Inheritance
categories:
- web
tags:
- rails
- ruby
---
----------

认识Rails STI很久了，但一直都是只在Model层动点皮毛。但STI的魅力远不止于此。在此记录下STI为Controller，route带来的便利。话不多说，假使我们有如下四个Model：
{% highlight ruby linenos %}
    # app/models/animal.rb
    class Animal < ActiveRecord::Base 
        self.inheritance_column = :race #指明STI字段，默认情况下为type,如果为model添加一个名为type的字段rails会智能的启用STI，如果不想启用需要用self.inheritance_column = :fake_column来指明。
    end
    
    class Lion < Animal
        def talk
            puts 'It means no worries for the rest of your days'
        end
    end
    
    class Meerkat < Animal
        def talk
            puts 'Hakuna Matata, what a wonderful phrase !'
        end        
    end 
    
    class WildBoar < Animal
        def talk
            puts 'Hakuna Matata! Ain't no passing craze'
        end
    end
{% endhighlight %}
以上为典型的STI model层的使用案例，那么如果要对animal生成CRUD，并且要求访问的路径不同，是否需要每个类别都要建立对应的controller，route呢？显然，这不够简洁。使用STI相关技巧即可实现用一个controller管理不同类别的animal，代码如下：
{% highlight ruby linenos %}
    # config/routes.rb
    resources :animals
    resources :lions, controller: 'animals', type: 'Lion' 
    resources :meerkats, controller: 'animals', type: 'Meerkat' 
    resources :wild_boars, controller: 'animals', type: 'WildBoar'
{% endhighlight %}
注意到type了吧，没错，这里就是STI在route中的魔法糖。当访问/lions的时候，实际上会被导向到animals controller中，并且rails会加一个名为type的参数到路径中：params[:type] = ‘Lion’。那么在animal中我们就可以这么写：
{% highlight ruby linenos %}
    class AnimalsController < ApplicationController 
        before_action :set_race
    
        def index 
            @animals = race_class.all 
        end
    
        # Code hidden for brivety
    
        private
    
        def set_race 
           @race = race 
        end
    
        def race 
            Animal.races.include?(params[:type]) ? params[:type] : "Animal"
        end
    
        def race_class 
            race.constantize 
        end
    
    end
{% endhighlight %}
这样，在访问不同类别animal的路径的时候，会显示对应类别的animal。以下是除index外其它方法的实现：
{% highlight ruby linenos %}
    # helpers/animals_helper.rb
    # 根据参数的不同动态的返回路径
    def sti_animal_path(race = "animal", animal = nil, action = nil)
      send "#{format_sti(action, race, animal)}_path", animal
    end
    
    def format_sti(action, race, animal)
      action || animal ? "#{format_action(action)}#{race.underscore}" : "#{race.underscore.pluralize}"
    end
    
    def format_action(action)
      action ? "#{action}_" : ""
    end
{% endhighlight %}
Index页面中的超链接：
{% highlight erb linenos %}
    <%= link_to 'Show', sti_animal_path(animal.race, animal) %>
{% endhighlight %}
animals controller继续改进：
{% highlight ruby linenos %}
    class AnimalsController < ApplicationController
      before_action :set_animal, only: [:show, :edit, :update, :destroy]
      before_action :set_race
    
      def index
        @animals = race_class.all
      end
    
      def show
      end
    
      def new
        @animal = race_class.new
      end
    
      private
      def set_animal
        @animal = race_class.find(params[:id])
      end
    
    end
{% endhighlight %}
这样在show页面中就可以自由的访问@animal的属性了
{% highlight erb linenos %}
    <h1>Hi, I'm a <%= @animal.race %>.</h1>
    <p>My name is <%= @animal.name %></p>
    <%= @animal.talk %>
    <!-- 返回到对应类别animal列表的超链接 -->
    <td><%= link_to "See all #{animal.race.pluralize}", sti_animal_path(animal.race) %></td> 
    <!-- 返回到所有类别animal列表的超链接 -->
    <%= link_to 'See all animals', sti_animal_path %>
    <%= link_to "New #{@race}", sti_animal_path(@race, nil, :new) %>
{% endhighlight %}
new页面：
{% highlight erb linenos %}
    <%= form_for(@animal) do |f| %>
      <% if @animal.errors.any? %>
        <div id="error_explanation">
          <h2><%= pluralize(@animal.errors.count, "error") %> prohibited this animal from being saved:</h2>
    
          <ul>
          <% @animal.errors.full_messages.each do |msg| %>
            <li><%= msg %></li>
          <% end %>
          </ul>
        </div>
      <% end %>
    
      <div class="field">
        <%= f.label :name %><br>
        <%= f.text_field :name %>
      </div>
    
      <div class="field">
          <%= f.label :race %><br>
          <%= f.select :race, Animal.races.map {|r| [r.humanize, r.camelcase]}, {}, disabled: @race != "Animal" %> 
      </div>
    
      <div class="field">
        <%= f.label :age %><br>
        <%= f.text_field :age %>
      </div>
    
      <div class="actions">
        <%= f.submit %>
      </div>
    <% end %>
{% endhighlight %}
Create的实现：
{% highlight ruby linenos %}
    class AnimalsController < ApplicationController
        def create 
            @animal = Animal.new(animal_params)
            if @animal.save
              redirect_to @animal, notice: "#{race} was successfully created."
            else
              render action: 'new'
            end
        end
    
        private
        def animal_params 
            params.require(race.underscore.to_sym).permit(:name, :race, :age) 
        end
    
    end
{% endhighlight %}
animal_params为Rails4的新特性，规定我们只接受特定的三个参数。

继续添加编辑和删除：
Index erb：
{% highlight erb linenos %}
      <td><%= link_to 'Edit', sti_animal_path(animal.race, animal, :edit) %></td>
      <td><%= link_to 'Destroy', sti_animal_path(animal.race, animal), method: :delete, data: { confirm: 'Are you sure?' } %></td>
{% endhighlight %}
Controller 中：
{% highlight ruby linenos %}
    def update
      if @animal.update(animal_params)
        redirect_to @animal, notice: "#{race} was successfully created."
      else
        render action: 'edit'
      end
    end
    
    def destroy
      @animal.destroy
      redirect_to animals_url
    end
{% endhighlight %}
最后是编辑页面：
{% highlight erb linenos %}
    #views/animals/edit.html.erb
    <h1>Editing <%= "#{@race}" %></h1>
    <%= render 'form' %>
    <%= link_to 'Back', sti_animal_path(@race) %>
{% endhighlight %}
