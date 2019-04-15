# Terraform Exercise 4 - Clean Up

The `destroy` feature of Terraform will delete all resources that it knows about, ie.
tracked in its state file. This is incredibly powerful, in that it allows you to reliably
clean up all the artifacts you've created and limit the possibility that you leave orphaned
resources (which could potentially cost money).

However, it is also very dangerous if used carelessly, as you could accidentially delete an
entire environment by mistake in minutes. You should always be exceptionally careful when
you use `destroy`.

Just as we've run `init`, `plan` and `apply`, you can similarly run:

```terraform destroy```

Terraform will now systematically delete all of its resources, and do so in the reverse order
of its dependency graph.
