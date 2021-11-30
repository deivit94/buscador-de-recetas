<template>
  <div>
      <!--  PANTALLA INICIO EMPIEZA ACÁ -->
      <div class="pantalla_inicio" v-show="screenActive == 'home'">
        <div class="container">
            <h1>Bienvenido a Recetas App</h1>
            <p>En esta app podrás crear tus propias recetas, guardarlas y buscarlas cuando las necesites</p>
            <div style="text-align: center">
              <button class="ingresar_button" type="button" v-on:click.prevent="screenActive = 'create'">Ingresar</button>
            </div>
        </div>
      </div>

      <!--  PANTALLA CREAR RECETA EMPIEZA ACÁ -->
      <div class="pantalla_crear" v-show="screenActive == 'create'">

          <!-- NAV -->
          <nav class="crear_nav">
              <button @click.prevent="screenActive = 'home'">Volver a Inicio</button>
              <h2>Crea una nueva receta</h2>
              <button @click.prevent="screenActive = 'search'">Buscador de recetas</button>
          </nav>

          <!-- FORM -->
          <div class="form_contenedor">
              <form class="form_crear">
                  <!-- NOMBRE DE LA RECETA -->
                  <input type="text"  placeholder="Nombre de la receta" v-model="foodName" required/>

                  <!-- INGREDIENTES -->
                  <input class="ingrediente_input" v-model="ingredient" type="text" placeholder="Ingresar aquí un ingrediente a la vez" />
                  <button class="agregar_ingrediente_button" @click.prevent="addIngredient">Agregar ingrediente</button>
                  <div class="ingredientes_agregados">
                      <span v-for="(ing, index) in ingredientsAdded" :key="ing.id">
                          <span class="ingrediente_span">{{ing}}<button @click="deleteIng(index)">x</button></span>
                      </span>
                  </div>

                  <!-- TIPO DE COMIDA -->
                  <div class="checkbox_contenedor">
                      <div class="div_dulce">
                          <label class="label_dulce" for="sweet">Dulce</label>
                          <input type="radio" id="sweet" name="food" value="dulce" v-model="foodType" required/>
                      </div>
                      <div class="div_salado">
                          <label class="label_salado" for="salty">Salado</label>
                          <input type="radio" id="salty" name="food" value="salado" v-model="foodType"/>
                      </div>
                  </div>

                  <!-- URL FOTO COMIDA -->
                  <input type="url" v-model="imageUrl" placeholder="URL de la foto" required/>

                  <!-- BUTTON GUARDAR RECETA -->
                  <button class="guardar_receta_button" @click.prevent="saveRecipe">Guardar receta</button>            
              </form>
          </div>
      </div>

      <!--  PANTALLA BUSCAR RECETAS EMPIEZA ACÁ -->
      <div class="pantalla_buscar" v-show="screenActive == 'search'">


          <!-- NAV -->
          <nav class="buscar_nav">
              <button @click.prevent="screenActive = 'create'">Crear nueva receta</button>
              <h2>Aquí podras buscar todas las recetas que creaste previamente</h2>
          </nav>

          <!-- BUSCADOR -->
          <div class="opciones_busqueda">
              <!-- NOMBRE DE LA RECETA O ALGUNA PALABRA DE LA RECETA -->
              <div class="nombre_receta_div">
                  <input type="text" v-model="foodNameSearch" placeholder="Nombre de la receta"/>
              </div>

              <!-- TIPO DE COMIDA SWITCH BUTTON -->
              <div class="tipo_comida_div">
                  <div class="switch_button_contenedor">
                      <button @click="foodTypeSearch = 'dulce'; sweetClick()" class="dulce_button"
                      :class="{ 'switch_button_seleccionado': sweetColor }">
                        Dulce
                      </button>
                      <button @click="foodTypeSearch = 'salado'; saltyClick()" class="salado_button"
                      :class="{ 'switch_button_seleccionado': saltyColor }">
                        Salado
                      </button>
                  </div>
              </div>

              <!-- INGREDIENTES DISPONIBLES -->
              <div class="ingredientes_disponibles_contenedor">
                  <div class="ingredientes_disponibles_div">
                      <span v-for="(ing) in allTheIngredients" :key="ing.id" @click="changeColor(ing)"
                      :class="{ 'ingrediente_seleccionado': ing.active }" class="ing_disponibles_span">

                            {{ing.name}}
                      </span>
                  </div>
              </div>
          </div>

          <!-- RESULTADOS DE LA BUSQUEDA -->
          <div class="resultados_busqueda_contenedor">
              <div v-for="(recipe) in filterRecipes" :key="recipe.id" class="receta_div">
                  <img v-bind:src="recipe.url" :alt="recipe.name">
                  <div class="receta_texto_contenedor">
                      <p class="name">{{recipe.name.charAt(0).toUpperCase() + recipe.name.slice(1)}}</p>
                      <p class="type">{{recipe.type.toUpperCase()}}</p>
                      <p class="ingredients_title">Ingredientes</p>
                      <p class="ingredients">{{recipe.ingredientsRecipe}}</p>
                  </div>
              </div>
          </div>
      </div>

  </div>
</template>

<script>

export default {
  name: 'entire-app',
  data() {
    return {
      screenActive: 'home',

      foodName: '',
      ingredient: '',
      foodType: '',
      imageUrl: '',

      ingredientsAdded: [],
      recipes: [],
      allTheIngredients: [],

      foodNameSearch: '',
      foodTypeSearch: '',

      sweetColor: false,
      saltyColor: false

    }
  },

  methods:{
    addIngredient(){
        if(this.ingredient.trim() != '' && this.ingredientsAdded.includes(this.ingredient.toLowerCase().trim()) == false) {

            this.ingredientsAdded.push(this.ingredient.toLowerCase().trim())

            if (this.allTheIngredients.some( (obj) => Object.values(obj).includes(this.ingredient.toLowerCase().trim())) == false){

                this.allTheIngredients.push({
                  name: this.ingredient.toLowerCase().trim(),
                  active: false
                })
            }

            this.ingredient = '';
        }
    },

    deleteIng(index){
        this.ingredientsAdded.splice(index,1)
    },

    saveRecipe(){
          if(this.foodName != '' && this.ingredientsAdded.length != 0 && this.foodType != '' && this.imageUrl != '' &&
            (this.imageUrl.toLowerCase().indexOf('http://') == 0 || this.imageUrl.toLowerCase().indexOf('https://') == 0)  ){

              this.recipes.push({
              name: this.foodName.toLowerCase(),
              ingredientsRecipe: this.ingredientsAdded.join(', '),
              type: this.foodType,
              url: this.imageUrl.toLowerCase()
              })

              this.foodName = ''
              this.ingredientsAdded = []
              this.foodType = ''
              this.imageUrl = ''
          }
    },

    changeColor(ing){
      ing.active = !ing.active;
    },

    sweetClick(){
      this.sweetColor = true;
      this.saltyColor = false;
    },
    saltyClick(){
      this.saltyColor = true;
      this.sweetColor = false;
    }

  },
  computed:{
     filterRecipes(){
        return this.recipes.filter((recipe) => {

          const foodName = this.foodNameSearch.toLowerCase().trim();
          const foodType = this.foodTypeSearch;
          const anyOfTheIngredientsIsTrue = this.allTheIngredients.some( (obj) => Object.values(obj).includes(true));

          const filterIngredientsObjects = this.allTheIngredients.filter((obj) => Object.values(obj).includes(true));
          const ingredientsTrueArr = filterIngredientsObjects.map((obj) => { return (Object.values(obj)).shift()});
          const ingredientsRecipe = recipe.ingredientsRecipe.split(", ")

          if(   recipe.name == foodName
                && foodType == ''                                                           //nombre exacto y el resto vacio
                && anyOfTheIngredientsIsTrue == false)
          { return true

          } else if(  recipe.name.split(" ").includes(foodName) == true
                      && foodType == ''                                                     //alguna palabra del nombre y el resto vacio
                      && anyOfTheIngredientsIsTrue == false)
          { return true

          } else if ( foodType == recipe.type                                                     //type exacto y el resto vacios
                      && anyOfTheIngredientsIsTrue == false
                      && foodName == '')
          { return true

          } else if ((ingredientsTrueArr.every((ing) => ingredientsRecipe.includes(ing))) == true       //incluye todos los ingredientes
                      && ingredientsTrueArr != 0                                                        //y el resto vacio
                      && foodName == ''
                      && foodType == '')
          { return true

          } else if ( recipe.name == foodName                                                      //nombre y type exactos
                      && foodType == recipe.type                                                   //ingredients todos false
                      && anyOfTheIngredientsIsTrue == false)
          { return true

          } else if ( recipe.name.split(" ").includes(foodName) == true                          //alguna palabra del nombre y type exactos
                      && foodType == recipe.type                                                 //ingredients todos false
                      && anyOfTheIngredientsIsTrue == false)
          { return true

          } else if ( recipe.name == foodName                                                             //nombre exacto
                      && (ingredientsTrueArr.every((ing) => ingredientsRecipe.includes(ing))) == true     //incluye todos los ingredientes
                      && foodType == '')                                                                  //type vacio
          { return true

          } else if ( recipe.name.split(" ").includes(foodName) == true                                     //alguna palabra del nombre
                      && (ingredientsTrueArr.every((ing) => ingredientsRecipe.includes(ing))) == true       //incluye todos los ingredientes
                      && foodType == '')                                                                    //type vacio
          { return true

          } else if ( recipe.type == foodType                                                               //type exacto
                      && (ingredientsTrueArr.every((ing) => ingredientsRecipe.includes(ing))) == true       //incluye todos los ingredientes
                      &&  foodName == '')                                                                   //nombre vacio
          { return true

          } else if ( recipe.name == foodName
                      && recipe.type == foodType                                                            //nombre y el resto exactos
                      && (ingredientsTrueArr.every((ing) => ingredientsRecipe.includes(ing))) == true)
          { return true

          } else if( recipe.name.split(" ").includes(foodName) == true
                   && recipe.type == foodType                                                             //alguna palabra del nombre
                   && (ingredientsTrueArr.every((ing) => ingredientsRecipe.includes(ing))) == true)       // y el resto exactos
          {
            return true
          }

        })
      }
  }

}
</script>

<style>
*{
  margin: 0;
}

body {
  background: #fde270;
}
body::-webkit-scrollbar {
  display: none;
}

.ingrediente_seleccionado {               /* Dynamic CSS Classes */
  background: #198754 !important;
  color: white !important;
}

.switch_button_seleccionado {
  background: #198754 !important;
  color: white !important;
  font-size: 1.9em !important;
}



.pantalla_inicio {                 /* EMPIEZA PANTALLA INICIO  */
  color: #ad4940;
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.pantalla_inicio h1 {
  font-size: 4em;
  text-align: center;
}
.pantalla_inicio p {
  font-size: 1.7em;
  color: #384653;
  font-style: italic;
  text-align: center;
}
.ingresar_button {
  background: #ad4940;
  color: whitesmoke;
  font-size: 1.4em;
  width: 30vw;
  height: 55px;
  border-radius: 5px;
  border: #552722 2px solid;
  cursor: pointer;
  margin-top: 30px;
}
.ingresar_button:hover {
  background: #f39890;
}


.pantalla_crear {             /* EMPIEZA PANTALLA CREAR RECETA  */
  color: #ad4940;
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
}
.crear_nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 100vw;
  height: 60px;
  background: whitesmoke;
  padding: 10px;
}
.crear_nav h2 {
  font-size: 2.5em;
}
.crear_nav button {
  border: #ad4940 2px solid;
  border-radius: 5px;
  height: 50px;
  width: 100px;
  font-size: 1em;
  font-weight: 600;
  color: #384653;
  cursor: pointer;
  background: white;
  /* text-align: center; */
}
.crear_nav button:hover {
  background: #ad4940;
  font-weight: 600;
  color: white
}
.form_contenedor {
  width: 100vw;
  display: flex;
  justify-content: center;
}
.form_crear {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: #ad4940;
  width: 35vw;
  border-radius: 15px;
  margin-top: 30px;
  margin-bottom: 20px;
  padding-bottom: 15px;
}
.form_crear input {
  margin-top: 30px;
  width: 70%;
  height: 30px;
  border-radius: 5px;
  border: #fde270 2px solid;
  font-size: 1.25em;
}
.agregar_ingrediente_button {
  background: #eee8d0;
  color: #384653;
  font-size: 1em;
  font-weight: 600;
  height: 45px;
  width: 35%;
  text-align: center;
  border-radius: 10px;
  cursor: pointer;
  border: #384653 1px solid;
  margin-top: 5px;
}
.agregar_ingrediente_button:active{
  background: #838383;
}
.ingredientes_agregados {
  margin-top: 15px;
  background: white;
  border: #fde270 2px solid;
  min-width: 70%;
  max-width: 90%;
  min-height: 100px;
  display: flex;
  flex-wrap: wrap;
}
.ingrediente_span{
  display: flex;
  /* flex-flow: row nowrap; */
  justify-content: center;
  align-items: center;
  background: #f3d343;
  height: 30px;
  font-size: 1.2em;
  color: black;
  border-radius: 5px;
  margin: 5px 10px;
  padding: 0 7px;
  cursor: pointer;
}
.ingrediente_span button{
  display: flex;
  justify-content:center;
  background: #dc3545;
  color: white;
  font-weight: bold;
  width: 20px;
  height: 19px;
  border: 1px solid #dc3545;
  margin-left: 3px;
  padding: 0;
  border-radius: 5px;
  cursor: pointer;
}
.checkbox_contenedor {
  width: 100%;
  display: flex;
  justify-content: space-evenly;
  color: white;
  margin-top: 15px;
}
.checkbox_contenedor input{
  margin: 0;
  padding: 0;
  width: 25px;
}
.div_dulce, .div_salado {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.label_dulce, .label_salado {
  font-size: 1.5em;
}

.guardar_receta_button {
  background: #fde270;
  color: #384653;
  font-size: 1.7em;
  font-weight: 500;
  height: 55px;
  width: 75%;
  margin-top: 40px;
  text-align: center;
  border-radius: 10px;
  cursor: pointer;
  border: #384653 1px solid;
}
.guardar_receta_button:hover {
  background: #f3d343;
  color: white;
  font-weight: 700;
}


.pantalla_buscar {                   /* EMPIEZA PANTALLA BUSCAR RECETAS  */
  min-height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
}
.buscar_nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  max-width: 100vw;
  height: 60px;
  background: #f5f5f5;
  padding: 10px;
  padding-right: 50px;
}
.buscar_nav h2 {
  font-size: 1.7em;
  font-style: italic;
  color: #384653;
}
.buscar_nav button {
  border: #ad4940 2px solid;
  border-radius: 5px;
  height: 65px;
  width: 140px;
  font-size: 1.1em;
  font-weight: 600;
  color: #384653;
  cursor: pointer;
  background: white;
}
.buscar_nav button:hover {
  background: #ad4940;
  font-weight: 600;
  color: white;
}
.opciones_busqueda {
  display: flex;
  flex-flow: row wrap;
  max-width: 100vw;
}
.nombre_receta_div{
  width: 50%;
  height: 100px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.nombre_receta_div input {
  width: 50%;
  height: 40px;
  font-size: 1.7em;
  border-radius: 5px;
  border: #384653 2px solid;
}
.tipo_comida_div {
  width: 50%;
  height: 100px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.switch_button_contenedor {
  width: 50%;
  height: 50px;
  display: flex;
  flex-flow: row;
  border-radius: 5px;
}
.dulce_button {
  background: white;
  color: black;
  border: none;
  width: 50%;
  cursor: pointer;
  font-size: 1.6em;
  border-top-left-radius: 5px;
  border-bottom-left-radius: 5px;
  border-top: black 2px solid;
  border-right: black 1px solid;
  border-bottom: black 2px solid;
  border-left: black 2px solid;
}
.salado_button {
  background: white;
  color: black;
  border: none;
  width: 50%;
  cursor: pointer;
  font-size: 1.6em;
  border-top-right-radius: 5px;
  border-bottom-right-radius: 5px;
  border-top: black 2px solid;
  border-right: black 2px solid;
  border-bottom: black 2px solid;
  border-left: black 1px solid;
}
.ingredientes_disponibles_contenedor {
  width: 100%;
  display: flex;
  justify-content: center;
  margin-bottom: 30px;
}
.ingredientes_disponibles_div {
  display: flex;
  flex-wrap: wrap;
  min-width: 80%;
  max-width: 90%;
  background: #eee8d0;
  min-height: 130px;
  margin-top: 15px;
  border: #384653 2px solid;
  border-radius: 5px;
}
.ingredientes_disponibles_div span{
  display: flex;
  justify-content: center;
  align-items: center;
  height: 40px;
  font-size: 1.3em;
  border-radius: 5px;
  border: black 1px solid;
  margin: 15px 10px;
  padding: 0 7px;
  cursor: pointer;
  user-select: none;
  background: white;
}
.resultados_busqueda_contenedor {
  display: flex;
  flex-flow: row wrap;
  justify-content: space-evenly;
  max-width: 100vw;
  min-height: 100vh;
  background: white;
  padding-top: 30px;
  padding-left: 30px;
  padding-right: 30px;
}

.receta_div {
  width: 220px;
  min-height: 250px;
  display: flex;
  flex-flow: column wrap;
  background:white;
  border-top-left-radius: 5px;
  border-top-right-radius: 5px;
  margin-right: 20px;
  margin-left: 20px;
  margin-bottom: 60px;
}
.receta_div img {
  width: 100%;
  height: 220px;
  margin-bottom: 10px;
  object-fit: cover;
  border-radius: 5px;
}
.receta_texto_contenedor {
  display: flex;
  flex-flow: column;
}
.receta_texto_contenedor .name{
  font-family: 'Roboto', sans-serif;
  font-weight: 400;
  font-size: 1.3em;
}
.receta_texto_contenedor .type {
  font-family: 'Roboto', sans-serif;
  font-weight: 300;
  font-size: 1.3em;
  margin-top: 8px;
}
.receta_texto_contenedor .ingredients_title {
  font-family: 'Roboto', sans-serif;
  font-style: italic;
  text-decoration: underline;
  font-weight: 400;
  font-size: 1em;
  margin-top: 12px;
}
.receta_texto_contenedor .ingredients {
  font-family: 'Roboto', sans-serif;
  font-weight: 400;
  font-size: 0.95em;
  margin-top: 3px;
}





@media (max-width: 1399.98px) {     /* MAX-WIDTH 1399.98px */
  .form_crear input {               /* PANTALLA CREAR RESPONSIVE */
    font-size: 1.1em;
  }
}

@media (max-width: 1199.98px) {     /* MAX-WIDTH 1199.98px */
  .form_crear input {               /* PANTALLA CREAR RESPONSIVE */
    font-size: 0.9em;
  }

  .buscar_nav h2 {                /* PANTALLA BUSCAR RECETAS RESPONSIVE  */
    font-size: 1.5em;
  }
  .nombre_receta_div input {
    font-size: 1.5em;
  }
}

@media (max-width: 991.98px) {     /* MAX-WIDTH 991.98px */
  .pantalla_inicio h1{          /* PANTALLA INICIO RESPONSIVE */
    font-size: 4.5em;
    color: #ad4940;
    width: 50vw;
    margin-bottom: 40px;
  }
  .pantalla_inicio p{
    font-size: 2em;
    width: 50vw;
  }
  .ingresar_button{
    font-size: 1.4em;
    width: 38vw;
    height: 60px;
    margin-top: 60px;
  }

  .form_crear{               /* PANTALLA CREAR RESPONSIVE */
    width: 50vw;
  }
  .form_crear input{
    width: 80%;
    height: 40px;
    border-radius: 5px;
    border: #fde270 2px solid;
    font-size: 1.2em;
  }
  .ingredientes_agregados {
    width: 80%;
  }

  .buscar_nav h2 {                /* PANTALLA BUSCAR RECETAS RESPONSIVE  */
    font-size: 1.3em;
  }
  .nombre_receta_div input {
    font-size: 1.3em;
  }

}

@media (max-width: 767.98px) {    /* MAX-WIDTH 767.98px */
  .pantalla_inicio h1{        /* PANTALLA INICIO RESPONSIVE */
    font-size: 3.5em;
    color: #ad4940;
    width: 100vw;
    margin-bottom: 50px;
  }
  .pantalla_inicio p{
    font-size: 1.7em;
    width: 80vw;
    margin: auto;
  }
  .ingresar_button{
    font-size: 1.4em;
    width: 45vw;
    height: 65px;
    margin-top: 100px;
  }

  .crear_nav{                /* PANTALLA CREAR RESPONSIVE */
    height: 160px;
  }
  .crear_nav h2 {
    text-align: center;
    width: 30%;
    font-size: 1.3em;
  }
  .crear_nav button {
    height: 70px;
    width: 100px;
    font-size: 1.05em;
  }
  .form_crear {
    width: 95%;
  }
  .form_crear input {
    width: 90%;
    height: 35px;
    font-size: 1.2em;
  }
  .ingredientes_agregados {
    width: 95%;
  }
  .ingrediente_span{
    height: 35px;
    font-size: 1.4em;
    margin: 10px 10px;
  }

  .opciones_busqueda {          /* PANTALLA BUSCAR RECETAS RESPONSIVE  */
    display: flex;
    flex-direction: column;
    max-width: 100vw;
  }
  .buscar_nav {
    padding: 10px;
  }
  .buscar_nav h2 {
    width: 50vw;
    line-height: 0.9em;
    font-size: 1.3em;
  }
  .buscar_nav button {
    width: 130px;
    font-size: 1.2em;
  }
  .nombre_receta_div{
    width: 100%;
    height: 100px;
  }
  .nombre_receta_div input {
    width: 90%;
    height: 50px;
    font-size: 1.7em;
  }
  .tipo_comida_div {
    width: 100%;
  }
  .switch_button_contenedor {
    width: 100%;
    height: 70px;
    justify-content: center;
  }
  .dulce_button {
    background: white;
    color: black;
    border: none;
    width: 40%;
    font-size: 1.6em;
    border-top-left-radius: 5px;
    border-bottom-left-radius: 5px;
    border-top: black 2px solid;
    border-right: black 1px solid;
    border-bottom: black 2px solid;
    border-left: black 2px solid;
  }
  .salado_button {
    background: white;
    color: black;
    border: none;
    width: 40%;
    font-size: 1.6em;
    border-top-right-radius: 5px;
    border-bottom-right-radius: 5px;
    border-top: black 2px solid;
    border-right: black 2px solid;
    border-bottom: black 2px solid;
    border-left: black 1px solid;
  }
  .ingredientes_disponibles_div {       
    max-width: 95%;
    min-height: 130px;
  }
  .ingredientes_disponibles_div span{
    height: 60px;
    font-size: 1.7em;
    margin: 20px 20px;
    padding: 0 12px;
  }
  .resultados_busqueda_contenedor {
    padding-top: 30px;
    padding-left: 0px;
    padding-right: 0px;
  }
  .receta_div {
    width: 80vw;
    margin-bottom: 130px;
  }
  .receta_div img {
    height: 80vw;
    margin-bottom: 15px;
    border-radius: 15px;
  }
  .receta_texto_contenedor .name{
    font-family: 'Roboto', sans-serif;
    font-weight: 400;
    font-size: 1.8em;
  }
  .receta_texto_contenedor .type {
    font-family: 'Roboto', sans-serif;
    font-weight: 300;
    font-size: 1.5em;
    margin-top: 8px;
  }
  .receta_texto_contenedor .ingredients_title {
    font-family: 'Roboto', sans-serif;
    font-style: italic;
    text-decoration: underline;
    font-weight: 300;
    font-size: 1.4em;
    margin-top: 12px;
  }
  .receta_texto_contenedor .ingredients {
    font-family: 'Roboto', sans-serif;
    font-weight: 300;
    font-size: 1.3em;
    margin-top: 3px;
  }
}
</style>



