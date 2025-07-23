## [[400_bad_request]]

## [[pom_errors]]

## [[kotlin_errors]]

-> ALWAYS CHECK HEADERS for a failing network call.
-> A **network call with no body** should have **content-length header value set as 0**, otherwise the call will fail. #webclient #PrematureCloseException