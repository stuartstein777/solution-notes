The problem is : We are given a number of coordinates and velocities of objects in the sky. Each second they move an amount determined by their velocity. e.g.
 
```
position=< 9,  1> velocity=< 0,  2>
position=< 7,  0> velocity=<-1,  0>
position=< 3, -2> velocity=<-1,  1>
position=< 6, 10> velocity=<-2, -1>
```

In one second the first object will be at `9, 3`, (the original `9,1` + `0,2`), the second will be at `6,0`, the third at `2,-1` and the fourth at `4,9`.
A second later they will be at `9,5`, `5,0`, `1,0`, `2,8`
etc

The problem input contains `338` of these nodes, with fairly large x and y starting co-ordinates.

If you were looking at these objects in the sky, after some unknown time in the future, and for one second, they will line up relative to each other in such a way that you would see them form a message.
 
What is this message?

## Read and parse the input.

This is just a regex and parse the resulting strings to integers.

```clojure
(defn parse-input [line]
  (let [[x y vx vy] (->> (re-seq #"^position=<(-?\s?\d+), (-?\s?\d+)> velocity=<(-?\s?\d+), (-?\s?\d+)>" line)
                         (first)
                         (rest)
                         (map #(Integer/parseInt (str/trim %))))]
    {:x x, :y y, :vx vx, :vy vy}))

(->> (slurp "inputs/2018-10")
     (str/split-lines)
     (map parse-input))
```

The destructuring of the parse in the `let binding` is really nice here.

## How do we know when the message will form ?

At any point we can figure out the size of the binding box that holds all our objects. This is just the largest x of all the objects minus the smallest x of all the objects and the same for y, we can also run the simulation forward and find the minimum size of the binding box. 
 
At some point the binding box will get to the smallest it will reach, then start getting bigger again. The message will likely form on or around this minima.
 
We need some code that takes a node and returns a new node with its position updated.

```clojure
(defn move [{:keys [x y vx vy]}]
  {:x (+ x vx)
   :y (+ y vy)
   :vx vx
   :vy vy})
```

And a function to calculate the bounding box:

```clojure
(defn get-bounding-box [nodes]
  (let [xs (map :x nodes)
        ys (map :y nodes)
        min-x (apply min xs)
        min-y (apply min ys)
        max-x (apply max xs)
        max-y (apply max ys)]
    [(- max-x min-x) (- max-y min-y)]))
```

To find the minimum bounding box we can just keep mapping `move` over the collection of `nodes`, and at each point calculate the bounding box. Compare this bounding box with the last box and stop when the box has gotten bigger.

```clojure
(defn solve [width height nodes]
  (let [next (map move nodes)
        [n-width n-height] (get-bounding-box next)]
    (if (> (* width height) (* n-width n-height))
      (recur n-width n-height next)
      [width height])))
```

Calling this with the input data:

```clojure
(->> (slurp "inputs/2018-10")
     (str/split-lines)
     (map parse-input)
     (solve Integer/MAX_VALUE Integer/MAX_VALUE)) => [61 9]
```

So the smallest the bounding box gets is `61x9`

### Printing the objects at this point.

To print the objects we can group them by the `y` value, then sort these groupings by the `y` value. At this point we will have all the rows to print.

There's a slight problem here, the relative positions aren't zero based. The smallest `x` might be in the hundreds (or higher). We can fix this by printing from the minimum x to the maximum x. e.g. say we had the vector of x's for a row of `[51 56 57 59 62 64]` then we can generate range of numbers from 51 to 64 and then replace them with either a ⭐or ⬛ depending if the value is in the original vector.

A function to turn a row into a string:

```clojure
(defn get-row-to-print [min-x max-x node-group]
  (let [xs (->> node-group second (map :x) set)
        rg (range min-x (inc max-x))]
    (str/join "" (map #(if (xs %) "⭐" "⬛") rg))))
```

A function to display the message:

```clojure
(defn display-message [nodes]
  (let [min-x (apply min (map :x nodes))
        max-x (apply max (map :x nodes))
        grouped-nodes (sort-by first (group-by :y nodes))]
    (->> grouped-nodes
         (map (partial get-row-to-print min-x max-x))
         (str/join "\n")
         (println))))
```

Update the `solve` function to draw the nodes at the point where the bounding box is minimized.

```
(defn solve [width height nodes]
  (let [next (map move nodes)
        [n-width n-height] (get-bounding-box next)]
    (if (> (* width height) (* n-width n-height))
      (recur n-width n-height next)
      (display-message nodes))))
```

Produces:

![image](https://user-images.githubusercontent.com/22107263/141010888-c2d3e048-d9ff-49bb-ae80-f8a16251680d.png)
