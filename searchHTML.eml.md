```eml

-- declares that this is the SignupForm module, which is how
-- other modules will reference this one if they want to import it and reuse its code.
module SignupForm where

-- Elm’s "import" keyword works mostly like "require" in node.js.
-- The “exposing (..)” option specifies that we want to bring the Html module’s contents
-- into this file’s current namespace, so that instead of writing out
-- Html.form and Html.label we can just use "form" and "label" without the "Html." prefix.
import Html exposing (..)

-- This works the same way; we also want to import the entire
-- Html.Events module into the current namespace.
import Html.Events exposing (..)

-- With this import we are only bringing a few specific functions into our
-- namespace, specifically "id", "type'", "for", "value", and "class".
import Html.Attributes exposing (id, type', for, value, class)

import StartApp
import Effects
import Regex exposing (..)
import String 

view actionDispatcher model =
    form
        [ id "signup-form" ]
        [ h1 [] [ text "EML Search Engine" ]
        , label [ for "username-field" ] [ text "username: " ]
        , input
            [ id "username-field"
            , type' "text"
            , value model.username
            , on "input" targetValue (\str -> Signal.message actionDispatcher { actionType = "SET_USERNAME", payload = str })
            ]
            []
        , div [ class "validation-error" ] [ text model.errors.username ]
        , label [ for "password" ] [ text "password: " ]
        , input
            [ id "password-field"
            , type' "password"
            , value model.password
            , on "input" targetValue (\str -> Signal.message actionDispatcher { actionType = "SET_PASSWORD", payload = str })
            ]
            []
        , div [ class "validation-error" ] [ text model.errors.password ]
        
        , div [ class "signup-button", onClick actionDispatcher { actionType = "VALIDATE", payload = "" } ] [ text "Sign Up!" ]
        , label [ for "keyword" ] [ text "keyword: " ]
        , input
            [ id "keyword-field"
            , type' "text"
            , value model.keyword
            , on "input" targetValue (\str -> Signal.message actionDispatcher { actionType = "SET_KEYWORD", payload = str })
            ]
            []
        , div [ class "validation-error" ] [ text model.errors.keyword ]
        , div [ class "search-button", onClick actionDispatcher { actionType = "SEARCH", payload = "" } ] [ text "Search"]
        , div [ class "validation-error" ] [ text model.errors.results ]
        ]


initialErrors =
    { username = "", password = "", keyword = "", results = ""}


getErrors model =
    { username =
        if model.username == "" then
            "Please enter a username!"
        else
            ""

    , password =
        if model.password == "" then
            "Please enter a password!"
        else
            ""
    , keyword = 
        if model.keyword == "" then
            "Please enter a keyword!"
        else
            ""
    , results =
        if model.keyword /= "" then
          -- "Not implemented!"
          "Results: " ++ String.join ", " (findl model.keyword crawlOut )
        else
            ""
    }
    


update action model =
    if action.actionType == "SEARCH" then 
        ( { model | errors = getErrors model }, Effects.none )        
    else if action.actionType == "VALIDATE" then
        ( { model | errors = getErrors model }, Effects.none )
    else if action.actionType == "SET_USERNAME" then
        ( { model | username = action.payload }, Effects.none )
    else if action.actionType == "SET_PASSWORD" then
        ( { model | password = action.payload }, Effects.none )
    else if action.actionType == "SET_KEYWORD" then
        ( { model | keyword = action.payload }, Effects.none )
    
    else
        ( model, Effects.none )


initialModel =
    { username = "ash", password = "asg"
    , keyword = "transform", results = "", errors = initialErrors }


app =
    StartApp.start
        { init = (initialModel, Effects.none)
        , update = update
        , view = view
        , inputs = []
        }

main =
    app.html
    
  
  
type alias Url       = String 
type alias Content   = String
type alias Wcontent  = (Url, Content)


crawlOut: List Wcontent
crawlOut = 
  [ 
    ("url1", "The life is to be lived to be transformed"),
    ("url2", "Monkeys can type a thousand words but translate?"), 
    ("url3", "Junk and junk all over"),
    ("url4", "Transform. Brand You. How will you challenge life?"),
    ("url5", "Do, trans-form, live")
  ]




devowel = replace All (regex "[aeiou]") (\_ -> "")
depunct = replace All (regex "[^a-zA-Z]") (\_ -> "")


find: String -> Content -> Bool
find pattern wtext = 
  let 
    wtext = wtext
            |> String.trim 
            |> String.toLower
            |> depunct
           
  in
    if String.contains (String.toLower pattern) wtext
    then 
      True
    else 
      False


findt: String -> Wcontent -> Maybe Url
findt pattern wc = 
  let 
    (url, content) = wc
  in
    if find pattern content
    then
      Just url
    else
      Nothing
      
findl: String -> List Wcontent -> List Url
findl pattern listres =
  listres
      |> List.filterMap (findt pattern)
  --  |> List.filter (\ s  -> (s /= Nothing)) 
  --  |> List.map    (\ s  -> Maybe.withDefault "na" s)
      
  -- List.filter (\ s -> (s /= "NA") ) (List.map (\ wc -> findt2 p wc) listres)
  
 
```