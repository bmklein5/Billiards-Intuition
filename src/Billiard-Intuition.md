



~~~~
// SHOT DATA
var shot1 = [49.576,49.768,51.739,49.479,49.236,49.236,48.991,50.194,49.911]
var shot2 = [85.839,86.108,87.691,85.671,86.231,85.404,85.123,86.005,84.746]
var shot3 = [19.799,20.103,17.589,19.799,20.304,20.304,20.304,19.799,20.153]
var shot4 = [46.37,46.452,46.857,45,44.712,45.285,45,45.567,44.712]
var shot5 = [17.223,17.849,16.699,17.745,17.223,17.745,18.263,17.223,17.952]
var shot6 = [90.83,87.11,93.18,87.267,87.67,85.989,87.481,89.882,87.416]
var shot7 = [59.681,59.52,59.549,57.995,58.155,58.314,61.213,59.681,58.314]
var shot8 = [13.387,13.658,12.844,12.953,13.496,14.036,13.496,12.953,13.55]
var shot9 = [33.345,32.984,33.465,32.619,32.619,32.619,33.024,33.425,33.225]
var shot10 = [191.475,170.018,-11.915 + 180,169.796,169.796,169.796,169.796,169.242,169.519]
var shot11 = [180.172,178.511,-3.548 + 180,180,179.427,178.854,178.282,178.282,181.489]
var shot12 = [10.869,11.585,10.869,11.31,11.31,11.86,11.86,11.31,11.53]
var shot13 = [30.626,28.855,30.922,27.924,27.474,27.924,28.369,30.541,28.635]
var shot14 = [91.46,79.311,266.73 - 180,73.093,76.261,80.396,100.705,94.326,75.125]
var shot15 = [62.585,62.464,62.279,60.673,61.213,61.991,62.117,62.61,60.945]

var allShots = [shot1, shot2, shot3, shot4, shot5, shot6, shot7, shot8,
               shot9, shot10, shot11, shot12, shot13, shot14, shot15]
editor.put("allShots", allShots)
~~~~



~~~~
var shotMeans = editor.get("shotMeans")
var allShots = editor.get("allShots")
var variance1 = listVar(allShots[1], shotMeans[1])
~~~~

~~~~
var calculateVar = function(shots, mean){
  var allVariances = map2(listVar, shots, mean)
  
  return allVariances
}

var shotMeans = editor.get("shotMeans")
var allShots = editor.get("allShots")

var variances = calculateVar(allShots, shotMeans)

editor.put("shotVariances", variances)
~~~~

~~~~
var recursiveDist = function(distributions, variances, count){

  var std = Math.pow(variances[count-1], 0.5)

  var currentDist = Gaussian({mu:0, sigma: std})
  
  var updatedDistributions = distributions.concat(currentDist)
  
  
  var newCount = count+1
  
  if(newCount>15){return updatedDistributions}
  else{return recursiveDist(updatedDistributions, variances, newCount)}
}


var shotVariances = editor.get("shotVariances")

var shotDistributions = recursiveDist([], shotVariances, 1)


editor.put("shotDistributions", shotDistributions)

~~~~

~~~~
var shotProbability = function(samples, distribution, lower, upper, count, madeShots){
  if(count < samples){
    var newCount = count + 1
    var value = sample(distribution)
    
    var validShot = ((value >= lower) && (value <= upper))
    
    var updatedMadeShots = validShot ? madeShots + 1 : madeShots
    
    return shotProbability(samples, distribution, lower, upper, newCount, updatedMadeShots)
  }
  else{return (madeShots / samples)}
}

var example = Gaussian({mu:0, sigma:1})

// var p = observe(example, 0)

var p = shotProbability(100000, example, -1, 1, 0, 0)
print(p)

editor.put("shotProbability", shotProbability)



~~~~

~~~~
var shotProbs = editor.get("shotProbability")


var shotLikelihoods = function(shotArray, count, allDistributions, allThresholds){
  var currentDist = allDistributions[count - 1]
  var currentLower = allThresholds[count-1][0]
  var currentUpper = allThresholds[count-1][1]
  
  var probability = shotProbs(100000, currentDist, currentLower, currentUpper, 0, 0)
  
  var updatedShotArray = shotArray.concat(probability)
  
  var newCount = count + 1
  
  if(newCount <= 15){
    return shotLikelihoods(updatedShotArray, newCount, allDistributions, allThresholds)
  }
  else{
    return updatedShotArray
  }
}

var allDistributions = editor.get("shotDistributions")
var allThresholds = editor.get("Thresholds")

var allShotProbs = shotLikelihoods([], 1, allDistributions, allThresholds)

editor.put("allShotProbs", allShotProbs)

print('')
print('')
print(mapIndexed(function(x, y){return [x+1, y]}, allShotProbs))
~~~~

~~~~
var avgConfidence = [9.000, 
                     7.778,
                     6.556,
                     5.444,
                     5.111,
                     4.667,
                     5.889,
                     4.889,
                     3.444,
                     3.667,
                     6.333,
                     3.667,
                     7.667,
                     4.500,
                     3.278];

var allShotProbs = editor.get("allShotProbs")

var realisticness = map2(function(x, y){return x / (y/10)}, allShotProbs, avgConfidence)

var dict = mapIndexed(function(x,y){return [x+1, y]}, realisticness)

print('')
print('')
print('')
var sortedDict = sort(dict, gt, function(x){return x[1]})

print(sortedDict)
~~~~
