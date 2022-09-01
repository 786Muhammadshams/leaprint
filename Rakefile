desc "Start up a development environment"
task :dev do
  if system("/usr/local/bin/podman container exists LEAPrintDev")
    sh "/usr/local/bin/podman start LEAPrintDev"
  else
    sh "/usr/local/bin/podman run --tz=local --name=LEAPrintDev --mount type=bind,source='#{pwd}',target=/app -t ubuntu:jammy"
    sh "/usr/local/bin/podman exec apt-get update"
    sh "/usr/local/bin/podman exec apt-get install -y libboost-dev libboost-locale-dev libpoco-dev libcairo2-dev curl"
  end
end

desc "Start the builder"
task :builder do
  if system("/usr/local/bin/podman container exists builder")
    sh "/usr/local/bin/podman start builder"
  else
    sh "/usr/local/bin/podman run --privileged --tz=local --name=builder -t quay.io/buildah/stable"
  end
end

desc "Copy files to builder"
task :copy do
  sh "podman exec builder rm -rf /rootfs"
  sh "podman exec builder mkdir /rootfs"
  sh "podman cp Dockerfile builder:/rootfs"
  sh "podman cp makeImage.sh builder:/rootfs"
  sh "podman cp Server/a.out builder:/rootfs"
  sh "podman cp Server/fonts builder:/rootfs"
end

desc "Build a docker image"
task :image do
  sh "podman exec -it builder /bin/bash -c 'cd /rootfs && ./makeImage.sh'"
end

desc "Push the new docker image to github packages (ghcr.io)"
task :push do
  puts "Enter image hash (from last \"rake image\"): "
  hash = STDIN.gets.strip

  login = `podman exec -it builder buildah login --get-login ghcr.io`
  if login != "mltucker"
    puts "Login with Github PAT (with write:packagers permission)"
    puts "This PAT can be generated in Github Developer Settings"
    sh "podman exec -it builder buildah login ghcr.io"
  end

  sh "podman exec -it builder buildah push #{hash} docker://ghcr.io/comrex/leaprint-server:latest"
end
