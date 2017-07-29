```haskell
-- |professions
data Ocupacao = Engenheiro | Professor | Gerente | Estudante
              deriving (Show, Read, Enum)

-- |letter grade
data Conceito = F | D | C | B | A
              deriving (Show, Read, Enum)

-- |'rank' converts the Conceito into a normalized rank value
rank :: String -> Double
rank co = (fromEnum' co') / (fromEnum' A)
  where 
    co' = read co :: Conceito
    fromEnum' = fromIntegral . fromEnum

-- |'binarize' parses a Ocupacao into a binary list
binarize :: String -> [Double]
binarize oc = [bool2double $ oc' ==  i | i <- [0..3]]
  where
    oc' = fromEnum (read oc :: Ocupacao)
    bool2double True  = 1.0
    bool2double False = 0.0

-- |'parseFile' parses a space separated file 
-- to a list of lists of Double
parseFile :: String -> [[Double]]
parseFile file = map parseLine (lines file)
  where
    parseLine l = concat $ toDouble (words l)
    toDouble [oc, co, grade] = [binarize oc, [rank co, read grade]]
```

```haskell
-- |'minkowski' distance
minkowski :: Double -> [Double] -> [Double] -> Double
minkowski p x y  = summation ** (1.0/p)
  where
    summation = sum $ zipWith (pointwiseDist) x y
    pointwiseDist xi yi = (abs (xi - yi)) ** p

-- |'euclidean' is just a special case of minkowski
euclidean :: [Double] -> [Double] -> Double
euclidean = minkowski 2

-- |'jaccard' distance
jaccard :: [Double] -> [Double] -> Double
jaccard x y = 1 - (sumProd / sumSum)
  where
    sumProd = sum $ zipWith (*) x y
    sumSum  = sum $ zipWith (binsum) x y

-- |'binsum' is a binary addition for Jaccard
binsum :: Double -> Double -> Double
binsum 1 _ = 1
binsum _ 1 = 1
binsum _ _ = 0

-- |'standardize' a vector
standardize :: [Double] -> [Double]
standardize x = map toCenter x
  where
    toCenter xi = (xi - mean x) / stdX
    mean x = (sum x) / (length' x)
    stdX  = sqrt varX
    varX  = mean $ map (\xi -> (xi - mean x) ** 2) x
    length' l = fromIntegral $ length l

-- |'maxminScale' scales vector to [0,1]
maxminScale :: [Double] -> [Double]
maxminScale x = map scale x
  where
    scale xi = (xi - minimum x) / (maximum x - minimum x)

-- |'norm' calculates the norm vector
norm :: [Double] -> [Double]
norm x = map (/nx) x
  where
    nx = sqrt . sum $ map (^2) x
```