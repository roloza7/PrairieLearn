# Executor

This image will be used to execute course code in isolation in production environments.

## Release process

This image is built and pushed to the container registry as `prairielearn/executor:GIT_HASH`, where `GIT_HASH` is the Git commit hash of the version of PrairieLearn the executor image was built from. This allows us to ensure a given version of PrairieLearn is always running with the correct executor image.

There are scripts in the `/tools` directory that will aid with the building and releasing of this image. They will automatically determine the correct tag from the underlying Git repository. The scripts should be run from the root of the repository.

To build and tag the image for local testing, run:

```sh
./tools/executor/build.js
```

To push the built image to the Docker and ECR registries, run:

```
./tools/executor/release.js
```

## In-dev testing

Build the image using the above `build-executor.sh` script. Make note of the name of the resulting image; you'll need this image name momentarily.

Start the container, using the version of the container that you noted above (the examples below use the Git hash `12345`). Note that we mount in the PrairieLearn `elements` directory so that the examples below can use PrairieLearn's elements:

```sh
docker run --rm -it -v /path/to/checked/out/PrairieLearn/elements/:/elements/ prairielearn/executor:12345
# Optionally, enable debug prints
docker run --rm -it -e DEBUG='*' -v /path/to/checked/out/PrairieLearn/elements/:/elements/ prairielearn/executor:12345
```

Once the container is running, paste the following and hit `Enter`:

```json
{"type":"core-element","directory":"pl-checkbox","file":"pl-checkbox","fcn":"prepare","args":["<pl-checkbox answers-name=\"ans\"><pl-answer correct=\"true\">correct</pl-answer></pl-checkbox>",{"params":{},"correct_answers":{}}]}
```

You should see the following printed to the console:

```json
{"data":{"params":{"ans":[{"key":"a","html":"correct"}]},"correct_answers":{"ans":[{"key":"a","html":"correct"}]}},"output":"","functionMissing":false}
```

Now, try to render the element:

```json
{"type":"core-element","directory":"pl-checkbox","file":"pl-checkbox","fcn":"render","args":["<pl-checkbox answers-name=\"ans\"><pl-answer correct=\"true\">correct</pl-answer></pl-checkbox>",{"editable":true,"params":{},"correct_answers":{},"submitted_answers":{},"panel":"question","partial_scoress":{}}]}
```

You should see the following printed to the console:

```json
{"data":"<script>\n    $(function(){\n        $('#pl-checkbox-528e174e-1589-4810-aa56-ae752717e777 [data-toggle=\"popover\"]').popover({\n            sanitize: false,\n            container: 'body',\n            template: '<div class=\"popover pl-checkbox-popover\" role=\"tooltip\"><div class=\"arrow\"></div><h3 class=\"popover-header\"></h3><div class=\"popover-body\"></div></div>',\n        });\n    });\n</script>\n\n\n<div class=\"d-block\">\n\n\n\n<span class=\"form-inline\">\n    <span id=\"pl-checkbox-528e174e-1589-4810-aa56-ae752717e777\" class=\"input-group pl-checkbox\">\n        <span> <small class=\"form-text text-muted\">Select all possible options that apply.</small> </span>\n        <span class=\"btn btn-sm \" role=\"button\" data-toggle=\"popover\" data-html=\"true\" title=\"Checkbox\" data-content=\"You must select at least one option. You will receive a score of 100% if you select all options that are true and no options that are false. Otherwise, you will receive a score of 0%.\" data-placement=\"auto\" data-trigger=\"focus\" tabindex=\"0\">\n            <i class=\"fa fa-question-circle\" aria-hidden=\"true\"></i>\n        </span>\n    </span>\n</span>\n\n\n\n\n\n\n</div>","output":"","functionMissing":false}
```