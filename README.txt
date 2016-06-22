var gulp = require('gulp'); <br>
var connect = require('gulp-connect'); // run a local dev server <br>
var open = require('gulp-open'); // open a URL in a web browser <br>
var browserify = require('browserify'); // Bundles JS <br>
var reactify = require('reactify') // Transforms React JSX to JS <br>
var source = require('vinyl-source-stream'); // use conventional text streams with gulp <br>
var concat = require('gulp-concat'); // concatenats files <br>
var lint = require('gulp-eslint'); // lint JS files, including JSX <br>
var clean = require('gulp-clean'); <br>
<br>
var config = { <br>
    port: 9005, <br>
    devBaseUrl: 'http://localhost', <br>
    paths: {
        html: './src/*.html',
        js: './src/**/*.js',
        images: './src/images/*',
        css: [
          'node_modules/bootstrap/dist/css/bootstrap.min.css',
          'node_modules/bootstrap/dist/css/bootstrap-theme.min.css'      
        ],
        dist: './dist',
        mainJs: './src/main.js'
    }
}

//start a local dev server
gulp.task('connect', function(){
    connect.server({
      root: ['dist']  ,
      port: config.port,
      base: config.devBaseUrl,
      livereload: true
    })    
});

// dependancy task will be run when this task runs
gulp.task('open', ['connect'], function(){
   gulp.src('dist/index.html') 
    .pipe(open({uri: config.devBaseUrl + ':' + config.port + '/'}));
});

gulp.task('html', function(){
    gulp.src(config.paths.html)
        .pipe(gulp.dest(config.paths.dist))
        .pipe(connect.reload());
});

gulp.task('js', function(){
    
    // written by husny to delete bundle.js if errors or at the time of rebuilding.
//    gulp.src(config.paths.dist + '/scripts/*.js', {read: false})
//		.pipe(clean());
    
   browserify(config.paths.mainJs)
    .transform(reactify)
    .bundle()
    .on('error', console.error.bind(console))
    .pipe(source('bundle.js'))
    .pipe(gulp.dest(config.paths.dist + '/scripts'))
    .pipe(connect.reload());
});

gulp.task('css', function(){
    gulp.src(config.paths.css)
        .pipe(concat('bundle.css')) // put inside this file
        .pipe(gulp.dest(config.paths.dist + '/css')); // // drop them in this location
});

gulp.task('images', function(){
    gulp.src(config.paths.images)
    .pipe(gulp.dest(config.paths.dist + '/images'))
    .pipe(connect.reload());
    
    //publish favicon
    gulp.src('./src/favicon.ico')
        .pipe(gulp.dest(config.paths.dist));
});

gulp.task('lint', function(){
   return gulp.src(config.paths.js)
    .pipe(lint({config: 'eslint.config.json'}))
    .pipe(lint.format());
});

gulp.task('watch', function(){
    gulp.watch(config.paths.html, ['html']);
    gulp.watch(config.paths.js, ['js', 'lint']);
})

gulp.task('default', ['html','css','images','js','lint','open','watch']); 
