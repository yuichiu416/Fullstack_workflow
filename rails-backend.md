
## Set up application_controller, users_controller, session_controller
1. app/controllers/application_controller.rb
   ex.
   ```rb
    class ApplicationController < ActionController::Base
        # protect_from_forgery with: :exception

        helper_method :current_user, :logged_in?

        def current_user
            @current_user ||= User.find_by(session_token: session[:session_token])
        end

        def logged_in?
            !!current_user
        end

        def login!(user)
            @current_user = user
            session[:session_token] = user.reset_session_token!
        end

        def logout!
            current_user.reset_session_token!
            session[:session_token] = nil
        end

        def require_logged_in
            unless current_user
            render json: { base: ['invalid credentials'] }, status: 401
            end
        end
    end
    ```
2. app/controllers/api/session_controller.rb
   ex.
   ```rb
    class Api::SessionsController < ApplicationController
        def create
            @user = User.find_by_credentials(params[:user][:username], params[:user][:password])
            if @user
            login!(@user)
            render 'api/users/show', status: 200
            else
            render json: ['Unable to log in with provided credentials'], status: 401
            end
        end

        def destroy
            if current_user
            logout!
            render json: {}
            else
            render json: {}, status: 404
            end
        end

        def info
            @user = User.find(params[:id])
            render 'api/users/info'
        end
    end
   ```
3. app/controllers/api/users_controller.rb
   ex.
   ```rb
    class Api::UsersController < ApplicationController
        def create
            @user = User.new(user_params)
            if @user.save
            @user.photo.attach(io: File.open("#{Rails.root}/app/assets/images/ronil.jpg"), filename: 'ronil.jpg')
            sleep(1)
            Deposit.create({user_id: @user.id, amount: 50000})
            login!(@user)
            render 'api/users/show', status: 200
            else
            render json: @user.errors.full_messages, status: 422
            end
        end

        def show
            @user = User.find(params[:id])
        end

        def update
            @user = User.find(params[:id]);
            if @user.update(user_params)
            render 'api/users/show', status: 200
            else
            render json: ["Please choose a file to upload"], status: 422
            end
        end

        def info
            @user = User.find(params[:id])
        end

        def portfolio
            @user = User.find(params[:id])
        end

        private

        def user_params
            params.require(:user).permit(:username, :email, :password, :photo)
        end
    end
   ```

## Set up the view json builders if necessary. 
**Note that only the extracted values will be passed to frontend**
ex. 
```rb
//app/views/api/pokemon/index.json.jbuilder
@pokemon.each do |poke|
    json.set! poke.id do
    json.extract! poke, :id, :name
    json.image_url image_path(poke.image_url)
    json.moves []
    json.item_ids []
    end
end

//app/views/api/pokemon/show.json.jbuilder
json.pokemon do
    json.extract! @pokemon, :id, :name, :attack, :defense, :moves, :poke_type, :item_ids
    json.image_url asset_path(@pokemon.image_url)
end

json.items do
    @pokemon.items.each do |item|
    json.set! item.id do
        json.partial! 'api/items/item', item: item
    end
    end
end

//app/views/api/item/show.json.jbuilder
json.partial! 'api/shared/items', item: @item

//app/views/api/item/_item.json.jbuilder
json.extract! item, :id, :name, :pokemon_id, :price, :happiness, :image_url
```
