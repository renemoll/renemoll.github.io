---
layout: post
title: Advent of Code 2020 in Haskell - Part 1
categories: programming haskell
date: 2021-01-18 16:01 -0600
---
In the last days of this year, I decided to solve the [Advent of Code 2020](https://adventofcode.com/). To make it even more of a challenge, I decided to solve the puzzles in a (for me) new programming language: Haskell. Why on Earth Haskell? I am curious about the type system and want to improve my functional programming, so what better way to stop reading about it and dive into it?

In this series of posts, I will cover my experience in learning Haskell and using it to solve the puzzles. At the end of the series, I will go back and apply what I learned to my solutions.

<!--more-->
---

# On the challenges

The Advent of Code consists out of 25 puzzles, each consisting out of two parts. There is a leaderboard for the people who solve the solution to quickest. 

Now I knew I was not going to be competing for the scoreboard. Mainly because I only found out about this challenge in the second half of December. Instead, I am using these challenges to improve my motivation to learn, read and experiment. The plan is to read one or two chapters in an online book and then solve one or two puzzles. Going back and forth between the two allows me to put what I read into practice, while to solve the puzzles will likely have to learn certain language constructs. I have had a good experience with learning algorithms in Java this way.

To learn, I am reading through [Learn You a Haskell for Great Good!](http://learnyouahaskell.com/), [Real World Haskell](http://book.realworldhaskell.org/read/), and (of course) a lot of StackOverflow discussions.

# Day 1

The overall story is that "I" am going on holiday. Only the journey is filled with puzzles that need to be solved to be able to continue. It starts even before we get to leave: [our expense account needs to be fixed](https://adventofcode.com/2020/day/1).

## Part 1

For the first part, I need to find two numbers from a list of numbers whose sum equals 2020. This can be solved easily with basic list comprehension, covered in one of the first chapters in [Learn You a Haskell for Great Good!](http://learnyouahaskell.com/chapters).

My challenge was with the way the input is provided, a file. Apparently, IO operations are a special class of functions within Haskell. Initially, I learned the basics from this [insightful](https://www.reddit.com/r/haskell/comments/8blpy6/how_to_read_line_by_line_in_haskell/dx7v43s/?utm_source=reddit&utm_medium=web2x&context=3) explaination. As explained in this [chapter](http://book.realworldhaskell.org/read/types-and-functions.html), functions without any side effects are part of a class named `pure`. Whereas functions with side effects are considered `impure`. IO related functions are considered impure and need special care. Consider what should happen when the application is unable to open a file, or there is no space available to write a file to, or a network connection is lost.

```haskell
get2020_2part_product :: [Int] -> Int
get2020_2part_product numbers = head [a*b | a <- numbers, b <- numbers, a + b == 2020]

main :: IO ()
main = do
  -- read the file line by line and convert each line to an Int
  numbers <- map (read :: String -> Int) . lines <$> readFile "day_01_input.txt"

  print $ get2020_2part_product numbers
```

# Part 2

For the second part, we solve the same challenge but now with 3 numbers:
```haskell
get2020_3part_product :: [Int] -> Int
get2020_3part_product numbers = head [a*b*c | a <- numbers, b <- numbers, c <- numbers, a + b + c == 2020]

main :: IO ()
main = do
  -- read the file line by line and convert each line to an Int
  numbers <- map (read :: String -> Int) . lines <$> readFile "day_01_input.txt"

  print $ get2020_3part_product numbers
```

And done!

## Lessons learned

* Reading data from file seems very easy and straight forward, `readFile` seems to do exactly what you want it to do. Combined with functions such as `lines` or `words` creates a powerful construct to read data from a file. The parsing that follows is more challenging.
* `<$>` is an infix synonym for `fmap`. Now, if I [understand it correctly](https://stackoverflow.com/questions/6824255/whats-the-point-of-map-in-haskell-when-there-is-fmap), it is used to convert from `IO String` to `String`. I will have to read up on `IO` later on to understand the necessity.
* The `.` indicates function composition, where the function on its left and right are combined into a single function. I use this in the file parsing expression to combine the reading of the file line by line and converting each line to an integer. 
* Typecasting. I was not expecting this would come up with the first assignment, though I find that the concept is rather elegant. Instead of casting a value (as done in languages as C/C++), one casts the function to use (ex: selecting the correct read function: `read :: String -> Int`.)

# Day 2

For the [second puzzle](https://adventofcode.com/2020/day/2), we need to help the people at the toboggan (or sledge) renting company before renting a sledge to get to the airport. The goal is to validate a list of passwords based on a condition given together with the password. The story goes that the file containing the passwords and criteria has been corrupted and we need to find out how many passwords are still valid.

So I need to read a file line by line, separate the condition from the password and then verify if the password passes the condition. I feel some more list comprehension coming up. 

## Part 1

Lets first have a look at the format to parse:
```
1-3 a: abcde
1-3 b: cdefg
2-9 c: ccccccccc
```
Each line consists out of two parts, the criteria before the colon and the password following it. The criteria itself is decoded as follows: the two numbers indicate the lower and higher limit for the number of occurrences of the following character in the password.
As such, the first and third entry is correct and the second one is corrupt.


For my solution I decided to split the solution into three steps: (1) parse a string into a data structure, (2) verify the password and (3) count the number of valid passwords.

In the previous puzzle, I found out how to parse simple numbers from a file. Now I need a way to parse a data structure from a file. I decided to use a simple [split](https://stackoverflow.com/questions/4978578/how-to-split-a-string-in-haskell) and store the result in a tuple (as I haven’t yet reached other data structures.):
```haskell
import Data.List.Split

parse :: [Char] -> ([Char], Char, Int, Int)
parse input = (password, check, low, high)
  where [crit, password] = splitOn ": " input
        [limits, checks] = splitOn " " crit
        [lows, highs] = splitOn "-" limits
        check = head checks
        low = read lows
        high = read highs

```

From here on it is straightforward to filter the password field on the character of interest and verify the number of occurrences. I could use a simple list comprehension, as before:
```haskell
validate :: ([Char], Char, Int, Int) -> Bool
validate (pw, c, low, high) = result
  where t = length [x | x <- pw, x == c]
        result = (t >= low) && (t <= high)
```

However, since my __intent__ is to filter the password, I want to use the actual [filter](http://learnyouahaskell.com/higher-order-functions#maps-and-filters) function:
```haskell
validateAmount :: ([Char], Char, Int, Int) -> Bool
validateAmount (pw, c, low, high) = result
  where t = length $ filter (==c) pw
        result = (t >= low) && (t <= high)
```

Combining both to read the file, parse it, and validate the passwords gives:
```haskell
main :: IO ()
main = do
  entries <- map parse . lines <$> readFile "day_02_input.txt"
  let result = length (filter validateAmount entries)
  print $ "Part 1: " ++ (show result)
```

## Part 2

For the second part of the challenge, it turns out our interpretation of the validation rule was incorrect. Instead of representing a range of valid occurrences, the numbers represent two positions within the password of which one (and only one) must contain that specific character. A simple variation to the validation function solves the puzzle:
```haskell
validatePosition :: Password -> Bool
validatePosition (Password pw c low high) = result
  where result = ((pw !! (low - 1)) == c) /= ((pw !! (high - 1)) == c)

main :: IO ()
main = do
  entries <- map parse . lines <$> readFile "day_02_input.txt"
  let result2 = length (filter validatePosition entries)
  print $ "Part 2: " ++ (show result2)
```

## Lessons learned

* While the use of tuples is straightforward, I would prefer an annotated data structure to indicate the intent of each field. I am confident this will come up later in the book(s).
* The `$` sign (not to be confused with `<$>`) (or [function application](http://learnyouahaskell.com/higher-order-functions#function-application)) makes the expression right of it take precedence over the expression left of it. Its main use seems to be to avoid parentheses and increase readability. Usually, I prefer to add parentheses as these give a clear visual indication of what is happening, and IDEs can highlight the start, end and content. But I will give this syntax a try.
* [Indexing a string](https://wiki.haskell.org/Cookbook/Lists_and_strings#Accessing_sublists) using the `!!` function is quite different from C/C++/Python.

# Day 3

[This puzzle](https://adventofcode.com/2020/day/3) is a take on pathfinding. Now that we have our sledge, we need to find a path from one edge of the map to another. Given a map (in a binary form) we have to count the number of trees (collisions) we would encounter from the top left following a specific heading until we reach the bottom of the map. 
The map itself is provided in a compressed form, where all rows are present with a limited number of columns. The columns are to be repeated into infinity until we reach the end of the map. With each step on the map, we ought to move three steps to the right and one to the bottom.

## Part 1

I believe this can be solved by loading the map into memory as a list of string and planning our movement over the map. Care needs to be taken when crossing the horizontal edge of the map, by resettings the position. From there on it is a matter of accessing the string indices and counting the trees. To trace the path, I want to generate a list of coordinates which represent the movement over the map, including overflow correction as follows:
| Step | Coordinate    |
| ---- | :-----------: |
| 0    | (0, 0)        |
| 1    | (3, 1)        |
| 2    | (6, 2)        |
| ...  | ...           |
| 11   | (2, 11)

Or, in code:
```haskell
-- Note: the result step is in (x,y)
step :: Int -> Int -> Int -> Int -> [(Int,Int)]
step right down xlimit ylimit = steps
  where steps = [(right*x `mod` ylimit, down*x) | x <- [0,1..xlimit], x*down<xlimit]
```
Note the second guard on the list generation expression, it ensures we stay on the map and avoid rounding errors due to division. 

From there on it is relatively straightforward to query the map, determine the object at each coordinate and sum the number of trees we encounter:
```haskell
encounteredTrees :: [String] -> Int -> Int -> Int
encounteredTrees input right down = trees
  where ylimit = length $ head input
        xlimit = length input
        steps = step right down xlimit ylimit
        trees = length $ filter (==True) [(input !! y !! x) == '#' | (x,y) <- steps]

main :: IO ()
main = do
  input <- lines <$> readFile "day_03_input.txt"

  let part1 = encounteredTrees input 3 1
  print $ "Part 1: " ++ (show part1)
```

The function may seem a bit too much, but it helps with the second part of the puzzle.

## Part 2

For the second part, we investigate different slopes to traverse the map. To generate a single solution, the resulting number of tree encounters is to be multiplied into a single number. Which is now trivial:
```haskell
main :: IO ()
main = do
  input <- lines <$> readFile "day_03_input.txt"

  let slopes = [(1,1), (3,1), (5,1), (7,1), (1,2)]
  let part2 = foldl (*) 1 [encounteredTrees input r d | (r,d) <- slopes]
  print $ "Part 2: " ++ (show part2)
```

# Day 4

Puzzle [number 4](https://adventofcode.com/2020/day/4) is about data parsing and validation. Upon arriving at the airport, we get stuck in a line awaiting an automated passport check, which is on the fritz. To speed things up, the challenge is to rewrite the passport check.

## Part 1

The first step of the challenge is to properly parse the input format. Each passport’s information is provided by several key-value pairs which may be spread over multiple consecutive lines. Passports are separated only by an empty line, it sounds like a wonderful system.
The key is to split the file on double newlines, splitting the text file into text blocks each represents the data of a single passport. Next, each text block is split into a list of key-value pairs.
```haskell
parse :: String -> (String, String)
parse input = let [key, value] = splitOn ":" input in (key, value)

...

input <- map (map parse . words) . splitOn "\n\n" <$> readFile "day_04_input.txt"
```

Next, we have to check if each passport contains the required fields. If not, the passport is considered invalid. It was a bit of puzzling until I realized I can extract the keys for each passport and match that with the required fields (using the lovely list comprehension technique):
```haskell
keys :: [(a, b)] -> [a]
keys passport = map fst passport

countValidFields :: [[(String, String)]] -> Int
countValidFields input = result
  where requiredFields = ["byr", "iyr", "eyr", "hgt", "hcl", "ecl", "pid"]
        result = length $ filter (==True) [and [field `elem` (keys passport) |  field <- requiredFields]  | passport <- input]

main :: IO ()
main = do
  input <- map (map parse . words) . splitOn "\n\n" <$> readFile "day_04_input.txt"
  print $ "Part 1: " ++ (show $ countValidFields input)
```

## Part 2

For the second part of the challenge, the actual values need to be validated, yippie. I find these text parsing exercises still cumbersome, I assume this is due to my lack of knowledge and experience on the matter, so this is good exercise. That being said, I came up with several specific validation functions which are called from a single entry point which selects the correct verification method based on the key name.
```haskell
verifyField :: String -> Maybe String -> Bool
verifyField _ Nothing = False
verifyField name (Just input) = case name of
  "byr" -> verifyLimit 1920 2002 input
  "iyr" -> verifyLimit 2010 2020 input
  "eyr" -> verifyLimit 2020 2030 input
  "hgt" -> verifyHeight input
  "hcl" -> verifyHairColour input
  "ecl" -> verifyEyeColour input
  "pid" -> verifyPassport input
  "cid" -> True
  _ -> False
```

A simple addition to the list magic and:
```haskell
countValidatedFields :: [[(String, String)]] -> Int
countValidatedFields input = result
  where requiredFields = ["byr", "iyr", "eyr", "hgt", "hcl", "ecl", "pid"]
        result = length $ filter (==True) [and [field `elem` (keys passport) && verifyField field (lookup field passport) |  field <- requiredFields]  | passport <- input]
```

Another puzzle solved! The complete solution can be found here: [day_04.hs](https://github.com/renemoll/advent_of_code_2020/blob/master/day_04.hs)

## Lessons learned

* Storing key-value pairs in a list of tuples seems a common pattern, see [`lookup`](https://hackage.haskell.org/package/base-4.14.1.0/docs/Prelude.html#v:lookup).
* A quick and dirty introduction to `Maybe`, since I have not reached the Monad chapter yet I will come back to that.
* I finally got to use guards and pattern matching for a function :).

# Day 5

I must say, our protagonist encounters a lot more programming challenges in the wild then I do. In [this case](https://adventofcode.com/2020/day/5), the challenge is to board the plane and find your seat. Only we lost our boarding pass. To locate our seat we got a file with all other boarding passes hence we have to find the missing one.

The challenge is to find the highest seat number from a list of encoded seat positions. The input seems to be very much like a binary format:
```
BFFFBBFRRR
FFFBBBFRRR
BBFFBBFRLL
```

The first seven characters encode the seat’s row and the last three characters the seat within the row. Combined, they form the seat number. Each F/B stands for front/back of the half of the total range of rows in the plane. So the first boarding pass starting with a B means we are in the first half of all the rows. From that half, we divide that in two and pick the next half based on the next character. And so forth until we cannot split the remaining range any further. The same logic is applied to the L/R character. Now replace F/B and L/R with zeros and ones and you are counting in [binary](https://en.wikipedia.org/wiki/Binary_number).

## Part 1

My first step is to write a converter function which converts a string into a number:
```haskell
fromBinary :: [Char] -> Int
fromBinary "" = 0
fromBinary input = sum [if c `elem` "BR" then 2^i else 0 | (c,i) <- zip (reverse input) [0,1..]]
```
Honestly, this snippet of code was not my first solution. Initially, I converted the characters to digits and used a recursive function to interpret each digit into a 2 complement component. It worked, but I like the one-liner better.

From here on it is trivial to implement the seat identifier formula:
```haskell
decodeSeat :: String -> Int
decodeSeat code = row * 8 + col
  where row = fromBinary $ take 7 code
        col = fromBinary $ drop 7 code
```

Using the usual input parsing routine we find the maximum seat identifier:
```haskell
main :: IO ()
main = do
  seats <- map decodeSeat . lines <$> readFile "day_05_input.txt"

  print $ "Part 1: " ++ (show $ maximum seats)
```

## Part 2

The next part of the challenge is to find your own seat. The encoding strategy allows for a large set of seat identifiers, however not all are valid! Seats at the front and back may not be present in the plane. However, both the seats next to us are present in the list. 

To find our seat we simply have to find the missing identifier from the available seats. By sorting the list we known all identifiers should have a difference of one between them unless there is one missing:
```haskell
findMissing :: [Int] -> Int
findMissing seats = head missing + 1
  where missing = [x | x:y:_ <- tails (sort seats), y-x > 1]
```
Note the `+ 1` as `x` represent the seat before ours.

## Leasons learned

The solution for this puzzle came quite quickly to me, allowing me to focus more on refactoring. I found it very enjoyable to reduce my code to a few lines and even increasing readablilty.
