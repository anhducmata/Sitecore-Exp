#Sitecore > Search > Solr
####1. Sitecore 9 SOLR search example
##### - First you must rebuid index from sitecore.
[![](https://scontent.fsgn3-1.fna.fbcdn.net/v/t34.0-12/26056941_1887455274917340_736584510_n.png?oh=348370b10e854da3260783628d20c2e0&oe=5A44F7D7)](https://scontent.fsgn3-1.fna.fbcdn.net/v/t34.0-12/26056941_1887455274917340_736584510_n.png?oh=348370b10e854da3260783628d20c2e0&oe=5A44F7D7)
##### -  Do Search
```csharp
public static SearchResults DoSearch(string searchTerm){

			var myResults = new SearchResults {
            	Results = new List<SearchResult>()
            };

			// Get the search index
			var searchIndex = ContentSearchManager.GetIndex("sitecore_master_index"); 

			// Build the search predicate
            var searchPredicate = GetSearchPredicate(searchTerm); 

			// Get a context of the search index
			 using (var searchContext = searchIndex.CreateSearchContext()) 
            {
				// Search the index for items which match the predicate
                var searchResults = searchContext
			.GetQueryable<SearchModel>()
			.Where(searchPredicate); 
 
                // This will get all of the results, which is not reccomended
                var fullResults = searchResults.GetResults();
 
                // Paging or ... searchResults.Page(1, 10).GetResults();
 
                foreach (var hit in fullResults.Hits)
                {
                    myResults.Results.Add(new SearchResult
                    {
                        Description = hit.Document.Description,
                        Title = hit.Document.ItemName,
                        Url = hit.Document.ItemUrl
                    });
                }
 
                return myResults;
            }
```
#####- GetSearchPredicate
```csharp
    public static Expression<Func<SearchModel, bool>> GetSearchPredicate(string searchTerm)
    {
        var predicate = PredicateBuilder.True<SearchModel>(); // Items which meet the predicate

        // Search the whole phrase - LIKE
        predicate = predicate.Or(x => x.DispalyName.Like(searchTerm)).Boost(1.2f);
        predicate = predicate.Or(x => x.PageDescription.Like(searchTerm)).Boost(1.2f);
        predicate = predicate.Or(x => x.Introduction.Like(searchTerm)).Boost(1.2f);
        predicate = predicate.Or(x => x.Maintext.Like(searchTerm)).Boost(1.2f);

        // Search the whole phrase - CONTAINS
        predicate = predicate.Or(x => x.DispalyName.Contains(searchTerm)).Boost(2.0f);
        predicate = predicate.Or(x => x.PageDescription.Contains(searchTerm)).Boost(2.0f);
        predicate = predicate.Or(x => x.Introduction.Contains(searchTerm)).Boost(2.0f);
        predicate = predicate.Or(x => x.Maintext.Contains(searchTerm)).Boost(2.0f);

        // Search the individual words
        foreach (var t in searchTerm.Split(' '))
        {
            var tempTerm = t;

            predicate = predicate.Or(x => x.DispalyName.Contains(t)).Boost(1.0f);
            predicate = predicate.Or(x => x.PageDescription.Contains(t)).Boost(1.0f);
            predicate = predicate.Or(x => x.Introduction.Contains(t)).Boost(1.0f);
            predicate = predicate.Or(x => x.Maintext.Contains(t)).Boost(1.0f);
        }

        return predicate;
    }
```
