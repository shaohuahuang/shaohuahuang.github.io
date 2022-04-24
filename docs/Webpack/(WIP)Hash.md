# Webpack Hash

## Hash
hash是跟整个项目的构建相关，只要项目里有文件更改，整个项目构建的hash值都会更改，并且全部文件都共用相同的hash值

## chunkhash
chunkhash，它根据不同的入口文件(Entry)进行依赖文件解析、构建对应的chunk，生成对应的哈希值。

简单来说这种是根据不同入口来配置的，比如vue-router、vuex、vue等公共入口文件，只要这些没有改变，那么他对应生成的js的hash值也不会改变。

## contenthash
contenthash主要是处理关联性，比如一个js文件中引入css，但是会生成一个js文件，一个css文件，但是因为入口是一个，导致他们的hash值也相同，所以当只有js修改时，关联输出的css、img等文件的hash值也会改变，这种情况下就需要contenthash了。

contenthash相对于chunkhash影响范围更小
每一个代码块（chunk）中的js和css输出文件都会独立生成一个hash，当某一个代码块（chunk）中的js源文件被修改时，只有该代码块（chunk）输出的js文件的hash会发生变化