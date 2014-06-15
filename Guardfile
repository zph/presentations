#
# Add files and commands to this file, like the example:
#   watch(%r{file/path}) { `command(s)` }
#
DIR = "surviving_large_unfamiliar_codebases"
guard :shell do
  watch(/#{DIR}\/(.*).md/) {|m| system("(cd #{DIR} && make reveal)") }
end
