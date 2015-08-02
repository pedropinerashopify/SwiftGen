#!/usr/bin/rake

def dev_dir
  xcode7 = `mdfind 'kMDItemCFBundleIdentifier == com.apple.dt.Xcode && kMDItemVersion == "7.*"'`.split("\n")
  raise "\n[!!!] You need to have Xcode 7.x installed to compile the SwiftGen tools\n\n" if xcode7.empty?
  %Q(DEVELOPER_DIR=#{xcode7.last.shellescape})
end

def build(scheme, install_root, install_dir)
  puts "\n== #{scheme} =="
  install_paths = "DSTROOT=#{install_root.shellescape} INSTALL_PATH=#{install_dir.shellescape}"
  sh %Q(#{dev_dir} xcodebuild -project SwiftGen.xcodeproj -scheme #{scheme} -sdk macosx install #{install_paths})
end

def bintask(scheme_tag)
  scheme = "swiftgen-#{scheme_tag}"
  desc "Build and install #{scheme} in ${install_root}${dir}.\n(install_root defaults to '.' and dir defaults to '/bin')"
  task scheme_tag, [:install_root, :dir] => :mkinstalldir do |_, args|
    args.with_defaults(:install_root => '.', :dir => '/bin')
    build(scheme, args.install_root, args.dir)
  end
end

###########################################################

task :mkinstalldir, [:install_root, :dir] do |_, args|
  args.with_defaults(:install_root => '.', :dir => '/bin')
  dir = "#{args.install_root}#{args.dir}"
  `[ -d #{dir.shellescape} ] || mkdir #{dir.shellescape}`
end

namespace :swiftgen do
  bintask 'l10n'
  bintask 'storyboard'
  bintask 'colors'
  bintask 'assets'
end

desc 'Build and install all executables in ${install_root}${dir}'
task :all, [:install_root, :dir] => %w(swiftgen:l10n swiftgen:storyboard swiftgen:colors swiftgen:assets)

task :default => [:all]

desc "Shortcut for `all[/usr/local,/bin]` (may need sudo).\nThis will install all SwiftGen executables in /usr/local/bin."
task :install do
  Rake::Task[:all].invoke('/usr/local','/bin')
end


###########################################################

namespace :playground do
  task :clean do
    sh 'rm -rf SwiftGen.playground/Resources'
    sh 'mkdir SwiftGen.playground/Resources'
  end
  task :assets do
    sh %Q(#{dev_dir} xcrun actool --compile SwiftGen.playground/Resources --platform iphoneos --minimum-deployment-target 7.0 --output-format=human-readable-text Tests/Assets/fixtures/Images.xcassets)
  end
  task :storyboard do
    sh %Q(#{dev_dir} xcrun ibtool --compile SwiftGen.playground/Resources/Wizzard.storyboardc --flatten=NO Tests/Storyboard/fixtures/Wizzard.storyboard)
  end
  task :localizable do
    sh %Q(#{dev_dir} xcrun plutil -convert binary1 -o SwiftGen.playground/Resources/Localizable.strings Tests/L10n/fixtures/Localizable.strings)
  end

  desc "Regenerate all the Playground resources based on the test fixtures.\nThis compiles the needed fixtures and place them in SwiftGen.playground/Resources"
  task :resources => %w(clean assets storyboard localizable)
end

###########################################################
