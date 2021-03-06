# roboservant

Automatically fuzz your servant apis in a contextually-aware way.

![CI](https://github.com/mwotton/roboservant/workflows/CI/badge.svg)


# why?

Servant gives us a lot of information about what a server can do. We
use this information to generate arbitrarily long request/response
sessions and verify properties that should hold over them.

# example

Our api under test:

```
newtype Foo = Foo Int
  deriving (Generic, Eq, Show, Typeable)
  deriving newtype (FromHttpApiData, ToHttpApiData)

type FooApi =
  "item" :> Get '[JSON] Foo
    :<|> "itemAdd" :> Capture "one" Foo :> Capture "two" Foo :> Get '[JSON] Foo
    :<|> "item" :> Capture "itemId" Foo :> Get '[JSON] ()
```

From the tests:

```
  assert "should find an error in Foo" . not
    =<< checkSequential (Group "Foo" [("Foo", RS.prop_sequential @Foo.FooApi Foo.fooServer)])
```

We have a server that blows up if the value of the int in a `Foo` ever gets above 10. Note:
there is no generator for `Foo` types: larger `Foo`s can only be made only by combining existing
`Foo`s with `itemAdd`. This is an important distinction, because many APIs will return UUIDs or
similar as keys, which make it impossible to cover a useful section of the state space without
using the values returned by the API


# why not servant-quickcheck?

[servant-quickcheck](https://hackage.haskell.org/package/servant-quickcheck)
is a great package and I've learned a lot from it. Unfortunately, as mentioned previously,
there's a lot of the state space you just can't explore without context: modern webapps are
full of pointer-like structures, whether they're URLs or database
keys/uuids, and servant-quickcheck requires that you be able to generate
these without context via Arbitrary.
