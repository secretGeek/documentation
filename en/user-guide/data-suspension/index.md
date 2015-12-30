# Data Suspension

Taking our classic ViewModel, we are going to decide what is important
to save upon application death/resume. We specifically do not save the
state of commands because they are recreated by the CTOR.

It's debatable if you were to keep the Search Results (maybe that's a
concern of your Akavache implementation)

But DEFINITELY you want to save the SearchQuery, as when that is
rehydrated it should restore the viewmodel to the exact state it was
in..

    [DataContract]
    public class SearchViewModel : ISearchViewModel
    {
        [IgnoreDataMember]
        public ReactiveList<SearchResults> SearchResults { get; set; }
    
    
        private string searchQuery;
        [DataMember]
        public string SearchQuery {
            get { return searchQuery; }
            set { this.RaiseAndSetIfChanged(ref searchQuery, value); }
     
        }
     
        [IgnoreDataMember]
        public ReactiveCommand<List<SearchResults>> Search { get; set; }
    
        [IgnoreDataMember] 
        public ISearchService SearchService { get; set; }
    }
