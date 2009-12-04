#!/usr/local/bin/macruby
framework 'Cocoa'
framework 'Quartz'
framework 'WebKit'

require 'optparse'
 
class VirtualBrowser
  attr_reader :view, :file_name
  
  def initialize
    NSApplication.sharedApplication.delegate = self
    @view     = WebView.alloc.initWithFrame([0, 0, 1024, 768])
    window    = NSWindow.alloc.initWithContentRect([0, 0, 1024, 768],
                                                styleMask:NSBorderlessWindowMask, 
                                                backing:NSBackingStoreBuffered, 
                                                defer:false)
 
    window.contentView = view
    setup_view_prefs
    view.frameLoadDelegate = self
    @captured = false
    @doc = PDFDocument.alloc.init
  end
  
  def fetch(url, number, file_name)
    0.upto(number - 1) do |i|
      page_url = NSURL.URLWithString("#{url}/##{i}")
      view.mainFrame.loadRequest NSURLRequest.requestWithURL(page_url)
      view.mainFrame.reload
      until @captured
        NSRunLoop.currentRunLoop.runUntilDate NSDate.date
      end
      @captured = false
    end
    
    filepath = File.expand_path(file_name)
    puts "saving #{filepath}"
    @doc.writeToFile(filepath, atomically:true)
  end
  
  def setup_view_prefs
    view.mediaStyle = 'screen'
    view.customUserAgent = 'Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_2; en-us) AppleWebKit/531.21.8 (KHTML, like Gecko) Version/4.0.4 Safari/531.21.10'
    view.preferences.autosaves = false
    view.preferences.shouldPrintBackgrounds = true
    view.preferences.javaScriptCanOpenWindowsAutomatically = false
    view.preferences.allowsAnimatedImages = false
    view.mainFrame.frameView.allowsScrolling = false
  end
  
  def webView(view, didFinishLoadForFrame:frame)
    NSRunLoop.currentRunLoop.runUntilDate NSDate.dateWithTimeIntervalSinceNow(0.2)
    save
  end
 
  def webView(view, didFailLoadWithError:error, forFrame:frame)
    raise "Failed to take snapshot: #{error.localizedDescription}"
    quit
  end
 
  def webView(view, didFailProvisionalLoadWithError:error, forFrame:frame)
    raise "Failed to take snapshot: #{error.localizedDescription}"
    quit
  end
  
  def quit
    NSApplication.sharedApplication.terminate(nil)
  end
  
  def save
    doc_view = setup_docview
    doc = PDFDocument.alloc.initWithData(doc_view.dataWithPDFInsideRect(doc_view.bounds))
    doc_view.unlockFocus
    @doc.insertPage(doc.pageAtIndex(0), atIndex:@doc.pageCount)
    @captured = true
  end
  
  def setup_docview
    doc_view = view.mainFrame.frameView.documentView
    doc_view.window.contentSize = [doc_view.bounds.size.width, doc_view.bounds.size.height]
    doc_view.frame = view.bounds
    doc_view.needsDisplay = true
    doc_view.lockFocus
    doc_view
  end
  
end


begin
  options = {}
  OptionParser.new do |opts|
    opts.banner = "Usage: #$0 [options] url [file]"

    opts.on("-n NUMBER", "--number NUMBER", Integer, "Number of slides") do |number|
      options[:number] = number
    end
  end.parse!
  
  raise OptionParser::ParseError unless options[:number]
  
  url       = ARGV.shift
  file_name = ARGV.shift || "slides.pdf"
  
  VirtualBrowser.new.fetch(url, options[:number], file_name)
rescue OptionParser::ParseError => e
  puts "Number is mandatory."
end