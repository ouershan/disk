disk
====var fs = require('fs'),
    path = require('path'),
    exec = require('child_process').exec;

var clearDir = function (dir, ext) {
    var files = fs.readdirSync(dir), ext = ext || '';
    files.forEach(function(file){
        var filepath = path.join(dir, file);
        try
        {
            var stat = fs.statSync(filepath);
            if (stat.isDirectory()) {
                clearDir(filepath, ext);
            }
            else if (stat.isFile())
            {
                 if (ext === 'all' || path.extname(filepath) == ext) {
                    try {
                        fs.unlinkSync(filepath);
                        //console.log('FILE %s deleted', filepath);
                    }
                    catch (e) {
                        console.log('%s delete fail, %j', filepath, e);
                    }
                 }
            }
        }
        catch (e)
        {
            return;
        }
    });

    if (ext === 'all') {
        fs.rmdirSync(dir);
        //console.log('PATH %s deleted', dir);
    }
};

var removeDede = function(sitepath) {
    var dedeAdmin = path.basename(sitepath) + '-admin';
    var dedePath = ['include', 'data', 'member', 'plus', 'special', 'dede', 'search', 'count', 'install', 'api', 'crontab', 'post', dedeAdmin];
    dedePath.forEach(function(dp){
        var abdp = path.join(sitepath, dp);
        if (path.existsSync(abdp)) {
            clearDir(abdp, 'all');
        }
    });
};

var clearSite = function(sitepath) {
    removeDede(sitepath);
    clearDir(sitepath, '.php');
};

if (process.argv.length > 2) {
    var sitepath = path.join('/web/wwwroot/s2', process.argv[2]);
    if (path.existsSync(sitepath)) {
        console.time('take');
        clearSite(sitepath);
        console.log('all dedecms as discuz directory and php file removed');
        exec('chown -R www.www ' + sitepath, function(error, stdout, stderr) {
            if (error !== null) {
                console.log('chown www.www: %j', error);
            }
            console.timeEnd('take');
        });
    }
    else
    {
        console.log('sitepath: %s is not exist(%s)', process.argv[2], sitepath);
    }
}
else {
    console.log('use: node check.js sitepath');
}

