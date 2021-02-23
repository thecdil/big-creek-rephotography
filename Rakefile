# Tasks for creating derivatives for spec-lumber blog, modified from collectionbuilder-sa 

require 'csv'

###############################################################################
# TASK: deploy
###############################################################################

desc "Build site with production env"
task :deploy do
  ENV["JEKYLL_ENV"] = "production"
  sh "bundle exec jekyll build"
end

###############################################################################
# Helper Functions
###############################################################################

$ensure_dir_exists = ->(dir) { if !Dir.exists?(dir) then Dir.mkdir(dir) end }

def prompt_user_for_confirmation message
  response = nil
  while true do
    # Use print instead of puts to avoid trailing \n.
    print "#{message} (Y/n): "
    $stdout.flush
    response =
      case STDIN.gets.chomp.downcase
      when "", "y"
        true
      when "n"
        false
      else
        nil
      end
    if response != nil
      return response
    end
    puts "Please enter \"y\" or \"n\""
  end
end

###############################################################################
# TASK: generate_derivatives
#
# this task assumes you put scanned objects in objects/archives folder
# newly added objects are then processed (filenames downcased, tifs to jpgs), copied to objects/
# and generates small and thumb image for image and pdf objects
#
###############################################################################

desc "Generate derivative image files from collection objects"
task :generate_derivatives, [:thumbs_size, :small_size, :density, :missing, :im_executable] do |t, args|
  args.with_defaults(
    :thumbs_size => "300x300",
    :small_size => "800x800",
    :density => "300",
    :missing => "true",
    #:im_executable => "magick"
    :im_executable => "convert"
  )

  # set the various directories to be used
  archives_dir = "objects/archives"
  objects_dir = "objects"
  thumb_image_dir = "objects/thumbs"
  small_image_dir = "objects/small"

  # Ensure that the output directories exist.
  [thumb_image_dir, small_image_dir].each &$ensure_dir_exists

  # support these file types
  EXTNAME_TYPE_MAP = {
    '.tiff' => :image,
    '.tif' => :image,  
    '.jpg' => :image,
    '.png' => :image,
    '.pdf' => :pdf
  }

  # Generate derivatives.
  Dir.glob(File.join([archives_dir, '*'])).each do |filename|
    # Ignore subdirectories.
    if File.directory? filename
      next
    end
    # Ignore readme.
    if filename == File.join([archives_dir, 'README.md'])
      next
    end

    # Get the lowercase filename without any leading path and extension.
    extname = File.extname(filename).downcase
    base_filename = File.basename(filename)[0..-(extname.length + 1)].downcase

    # Determine the file type and skip if unsupported.
    file_type = EXTNAME_TYPE_MAP[extname]
    if !file_type
      access_filename=File.join([objects_dir, "#{base_filename}#{extname}"])
      puts "Skipping conversion of file with unsupported extension: #{extname}."
      if args.missing == 'false' or !File.exists?(access_filename)
        puts "Copying #{filename} to objects dir for access as #{access_filename}."
        system("cp #{filename} #{access_filename}")
      end
      next
    end

    # Define the file-type-specific ImageMagick command prefix.
    cmd_prefix =
      case file_type
      when :image then "#{args.im_executable} #{filename}"
      when :pdf then "#{args.im_executable} -density #{args.density} #{filename}[0]"
      end

    # Generate or copy the access image.
    if extname == ".tif" || extname == ".tiff"
      access_filename=File.join([objects_dir, "#{base_filename}.jpg"])
      if args.missing == 'false' or !File.exists?(access_filename)
        puts "Creating #{access_filename}"
        system("#{cmd_prefix} -flatten #{access_filename}")
      else
        puts "Skipping existing file"
      end
    else
      access_filename=File.join([objects_dir, "#{base_filename}#{extname}"])
      if args.missing == 'false' or !File.exists?(access_filename)
        puts "Copying #{filename} to objects dir for access as #{access_filename}."
        system("cp #{filename} #{access_filename}")
      else
        puts "Skipping existing file"
      end
    end

    # Generate the thumb image.
    thumb_filename=File.join([thumb_image_dir, "#{base_filename}_th.jpg"])
    if args.missing == 'false' or !File.exists?(thumb_filename)
      puts "Creating: #{thumb_filename}";
      system("#{cmd_prefix} -resize #{args.thumbs_size} -flatten #{thumb_filename}")
    end

    # Generate the small image.
    small_filename = File.join([small_image_dir, "#{base_filename}_sm.jpg"])
    if args.missing == 'false' or !File.exists?(small_filename)
      puts "Creating: #{small_filename}"
      system("#{cmd_prefix} -resize #{args.small_size} -flatten #{small_filename}")
    end
  end
end


###############################################################################
# TASK: rename_by_csv
#
# read csv, rename files
#
###############################################################################

desc "rename objects using csv"
task :rename_by_csv do
  
  # set the various directories to be used
  raw_dir = "objects/raw/"
  archives_dir = "objects/archives/"
  csv_file = "_data/photograph_metadata_images.csv"
  filename_current = "filename_original"
  filename_new = "filename" 

  # open csv file
  csv_text = File.read(csv_file, :encoding => 'utf-8')
  csv_contents = CSV.parse(csv_text, headers: true)

  # iterate on csv rows
  csv_contents.each do |item|
    if item[filename_current]
      name_old = raw_dir + item[filename_current]
    else
      puts "no current filename"
      break
    end
    if item[filename_new]
      name_new = archives_dir + item[filename_new]
    else
      puts "no new filename"
      break
    end
    puts "copying: '#{name_old}' to '#{name_new}'"
    cp_cmd = "cp \"#{name_old}\" \"#{name_new}\""
    system( cp_cmd )
  end

end
