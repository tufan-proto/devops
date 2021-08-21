# dev-ops

A CI/CD pipeline is a pre-requisite for most modern
internet delivered software. 

This repo implements a CD pipeline for [`backstage`](https://backstage.io) on Azure+Kubernetes. By design,
the pipeline is implemented to be plug-able to the code
repository.

This is done for a few reasons:
1. Document the CD pipeline for human understanding
2. Strong change-management semantics
3. Allows replacing the architecture if necessary

## About this template
The documentation is implemented as a markdown file
with embedded scripts - a form of literate programming.
The document is the "source of truth" and the scripts
are generated from the document. This allows the documentation and the scripts to stay in sync.

Typical CD infrastructure can get to being quite complex
in the different tools used and the number of independent configuration values to be fiddled with. By maintaining
it as a narrative, we can provide a rich understanding, 
with images, links and presented as an HTML page, while
still writing executable scripts.


## Development
In production, a template is designed to be self-contained. Outside of nodejs, the CD template
does not itself need any other dependencies. However
the specific architecture and it's implementation will
impose dependency requirements. For example, in the reference implementation, we'll require `azure-cli`,
`kubectl` (the kubernetes CLI), `gh` (the github CLI),
`helm`.

The workflow for developing a template like this:

```
git clone https://github.com/acuity-sr/dev-ops
# edit the various markdown files in ./deployment.md
npm run build
git add .
git push
```

visit https://raw.github.com/acuity-sr/dev-ops