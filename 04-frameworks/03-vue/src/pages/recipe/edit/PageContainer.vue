<template>
  <recipe-edit-page
    :recipe="recipe"
    :recipe-error="recipeError"
    :on-update-recipe="onUpdateRecipe"
    :on-save="onSave"
    :on-add-ingredient="onAddIngredient"
    :on-remove-ingredient="onRemoveIngredient"
  />
</template>

<script lang="ts">
import Vue, { PropOptions } from "vue";
import RecipeEditPage from "./Page.vue";
import { createEmptyRecipe, createEmptyRecipeError } from "./viewModel";
import { mapRecipeApiToVM, mapRecipeVMToApi } from "./mapper";
import { fetchRecipeById, save } from "../../../rest-api/api/recipe";
import { validations } from "./validations";

export default Vue.extend({
  name: "RecipeEditContainer",
  components: { RecipeEditPage },
  props: {
    id: { required: true } as PropOptions<String>,
  },
  data() {
    return {
      recipe: createEmptyRecipe(),
      recipeError: createEmptyRecipeError(),
    };
  },
  beforeMount() {
    // Justo antes de montar el componente
    // Lab: validar que sea un número correcto o sino snackbar
    const id = Number(this.id || 0);
    fetchRecipeById(id)
      .then((recipe) => {
        this.recipe = mapRecipeApiToVM(recipe);
      })
      .catch((error) => console.log(error));
  },
  methods: {
    onUpdateRecipe(field: string, value: string) {
      this.recipe = {
        ...this.recipe,
        [field]: value,
      };
      this.validateRecipeField(field, value);
    },
    onSave() {
      validations.validateForm(this.recipe).then((result) => {
        if (result.succeeded) {
          const recipe = mapRecipeVMToApi(this.recipe);
          save(recipe)
            .then((message) => {
              this.$store.dispatch("setSnackbar", {
                text: message,
              });
              this.$router.back();
            })
            .catch((error) =>
              this.$store.dispatch("setSnackbar", {
                color: "error",
                text: "Failed to save the recipe",
              })
            );
        } else {
          this.$store.dispatch("setSnackbar", {
            color: "error",
            text: "Please, complete all required fields",
          });
        }
      });
    },
    onAddIngredient(ingredient: string) {
      console.log(`Add ${ingredient}`);
      if (ingredient !== "") {
        this.recipe = {
          ...this.recipe,
          ingredients: [...this.recipe.ingredients, ingredient],
        };
        this.validateRecipeField("ingredients", this.recipe.ingredients);
      } else {
        this.$store.dispatch("setSnackbar", {
          color: "error",
          text: "Enter an ingredient to add, please",
        });
      }
    },
    onRemoveIngredient(ingredient: string) {
      console.log(`Delete ${ingredient}`);
      this.recipe = {
        ...this.recipe,
        ingredients: this.recipe.ingredients.filter(
          (item) => item !== ingredient
        ),
      };
      this.validateRecipeField("ingredients", this.recipe.ingredients);
    },
    validateRecipeField(field, value) {
      validations.validateField(field, value).then((result) => {
        this.updateRecipeError(field, result);
      });
    },
    updateRecipeError(field, result) {
      this.recipeError = {
        ...this.recipeError,
        [field]: result,
      };
    },
  },
});
</script>
