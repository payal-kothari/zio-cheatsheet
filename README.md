# ZIO Cheat Sheet

- This is based on [ZIO](https://github.com/scalaz/scalaz-zio) 1.0-RC4.
- For simplicity, ZIO environment has been omitted but all the functions also work with the form `ZIO[R, E, A]`.

## Creating effects

| Name                                  | Given                                                        | To                     |
| --------------------------------------- | ------------------------------------------------------------ | ---------------------- |
| IO.succeed                              | `A`                                                          | `IO[Nothing, A]`       |
| IO.succeedLazy                          | `=> A`                                                       | `IO[Nothing, A]`       |
| IO.fail                                 | `E`                                                          | `IO[E, Nothing]`       |
| IO.interrupt                            |                                                              | `IO[Nothing, Nothing]` |
| IO.die                                  | `Throwable`                                                  | `IO[Nothing, Nothing]` |
| IO.effect <br> IO.apply <br> Task.apply | `=> A`                                                       | `IO[Throwable, A]`     |
| IO.effectTotal <br> UIO.apply           | `=> A`                                                       | `IO[Nothing, A]`       |
| IO.effectAsync                          | `((IO[E, A] => Unit) => Unit)`                               | `IO[E, A]`             |
| IO.fromEither                           | `Either[E, A]`                                               | `IO[E, A]`             |
| IO.fromFiber                            | `Fiber[E, A]`                                                | `IO[E, A]`             |
| IO.fromFuture                           | `ExecutionContext => Future[A]`                              | `IO[Throwable, A]`     |
| IO.fromOption                           | `Option[A]`                                                  | `IO[Unit, A]`          |
| IO.fromTry                              | `Try[A]`                                                     | `IO[Throwable, A]`     |
| IO.bracket                              | `IO[E, A` <br> `A => IO[Nothing, Unit]` <br> `A => IO[E, B]` | `IO[E, B]`             |
| IO.when                                 | `Boolean` <br> `IO[E, _]`                                    | `IO[E, Unit]`          |
| IO.whenM                                | `IO[E, Boolean]` <br> `IO[E, _]`                             | `IO[E, Unit]`          |

## Transforming effects

| Name          | From                    | Given                 | To                      |
| --------------- | ----------------------- | --------------------- | ----------------------- |
| map             | `IO[E, A]`              | `A => B`              | `IO[E, B]`              |
| const           | `IO[E, A]`              | `B`                   | `IO[E, B]`              |
| unit            | `IO[E, A]`              |                       | `IO[E, Unit]`           |
| `>>=` (flatmap) | `IO[E, A]`              | `A => IO[E1, B]`      | `IO[E1, B]`             |
| flatten         | `IO[E, IO[E1, A]]`      |                       | `IO[E1, A]`             |
| bimap           | `IO[E, A]`              | `E => E2`<br>`A => B` | `IO[E2, B]`             |
| mapError        | `IO[E, A]`              | `E => E2`             | `IO[E2, A]`             |
| flatmapError    | `IO[E, A]`              | `IO[Nothing, E2]`     | `IO[E2, A]`             |
| sandbox         | `IO[E, A]`              |                       | `IO[Cause[E], A]`       |
| flip            | `IO[E, A]`              |                       | `IO[A, E]`              |
| absolve         | `IO[E, Either[E, A]]`   |                       | `IO[E, A]`              |
| get             | `IO[Nothing, Option[A]` |                       | `IO[Unit, A]`           |
| toFuture        | `IO[Throwable, A]`      |                       | `IO[Nothing, Future[A]` |

## Recover from errors

| Name        | From       | Given                                | To                          |
| ------------- | ---------- | ------------------------------------ | --------------------------- |
| either        | `IO[E, A]` |                                      | `IO[Nothing, Either[E, A]]` |
| option        | `IO[E, A]` |                                      | `IO[Nothing, Option[A]]`    |
| run           | `IO[E, A]` |                                      | `IO[Nothing, Exit[E, A]]`   |
| `<>` (orElse) | `IO[E, A]` | `IO[E2, A1]`                         | `IO[E2, A1]`                |
| orElseEither  | `IO[E, A]` | `IO[E2, B]`                          | `IO[E2, Either[A, B]]`      |
| fold          | `IO[E, A]` | `E => B`<br>`A => B`                 | `IO[Nothing, B]`            |
| foldM         | `IO[E, A]` | `E => IO[E2, B]`<br>`A => IO[E2, B]` | `IO[E2, B]`                 |
| catchAll      | `IO[E, A]` | `E => IO[E2, A1]`                    | `IO[E2, A1]`                |
| catchSome     | `IO[E, A]` | `PartialFunction[E, IO[E1, A1]]`     | `IO[E1, A1]`                |
| retry         | `IO[E, A]` | `Schedule[E1, S]`                    | `ZIO[Clock, E1, A]`         |

## Combining effects + parallelism

| Name             | From       | Given                                            | To                               |
| ------------------ | ---------- | ------------------------------------------------ | -------------------------------- |
| IO.foldLeft        |            | `Iterable[A]` <br> `S` <br> `(S, A) => IO[E, S]` | `IO[E, S]`                       |
| IO.foreach         |            | `Iterable[A]` <br> `A => IO[E, B]`               | `IO[E, List[B]]`                 |
| IO.foreachPar      |            | `Iterable[A]` <br> `A => IO[E, B]`               | `IO[E, List[B]]`                 |
| IO.foreachParN     |            | `Long` <br> `Iterable[A]` <br> `A => IO[E, B]`   | `IO[E, List[B]]`                 |
| IO.forkAll         |            | `Iterable[IO[E, A]]`                             | `IO[Nothing, Fiber[E, List[A]]]` |
| fork               | `IO[E, A]` |                                                  | `IO[Nothing, Fiber[E, A]]`       |
| `<*>` (zip)        | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, (A, B)]`                 |
| `*>` (zipRight)    | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, B]`                      |
| `<*` (zipLeft)     | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, A]`                      |
| `<&>` (zipPar)     | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, (A, B)]`                 |
| `&>` (zipParRight) | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, B]`                      |
| `<&` (zipParLeft)  | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, A]`                      |
| race               | `IO[E, A]` | `IO[E1, A1]`                                     | `IO[E1, A1]`                     |
| raceAll            | `IO[E, A]` | `Iterable[IO[E1, A1]]`                           | `IO[E1, A1]`                     |
| raceEither         | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, Either[A, B]]`           |

## Finalizers

| Name        | From       | Given                      | To         |
| ------------- | ---------- | -------------------------- | ---------- |
| ensuring      | `IO[E, A]` | `UIO[_]`                   | `IO[E, A]` |
| onError       | `IO[E, A]` | `Cause[E] => UIO[_]`       | `IO[E, A]` |
| onInterrupt   | `IO[E, A]` | `UIO[_]`                   | `IO[E, A]` |
| onTermination | `IO[E, A]` | `Cause[Nothing] => UIO[_]` | `IO[E, A]` |

## Timing

| Name   | From       | Given            | To                          |
| -------- | ---------- | ---------------- | --------------------------- |
| IO.never |            |                  | `IO[Nothing, Nothing]`      |
| IO.sleep |            | `Duration`       | `ZIO[Clock, Nothing, Unit]` |
| delay    | `IO[E, A]` | `Duration`       | `IO[E, A]`                  |
| timeout  | `IO[E, A]` | `Duration`       | `ZIO[Clock, E, Option[A]]`  |
| forever  | `IO[E, A]` |                  | `IO[E, Nothing]`            |
| repeat   | `IO[E, A]` | `Schedule[A, B]` | `ZIO[Clock, E, B]`          |